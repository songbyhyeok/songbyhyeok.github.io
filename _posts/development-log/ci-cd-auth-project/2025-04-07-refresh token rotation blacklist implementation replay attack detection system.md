---
title: 9 - Refresh Token Rotation, Blacklist, ReplayAttack 감지 시스템 구현
categories: ci-cd-auth-project
---

# 개요
지난 편의 주제인 Access & Refresh Flow를 바탕으로 JWT 인증 시스템을 구현했다. 그래서 코드 중심으로 간단히 소개하려고 한다.  

<br>

## 코드 흐름
<img src="/assets/image/projects/cicd-auth/2025-04-07-implementation of jwt security system using access token and refresh token - 1.png" alt="empty" style="width: 100%; height: auto;">  

1. 클라이언트가 인증 이후 요청을 보내면 Spring Security Filter Chain에 등록된 JWT Filter가 호출이 된다.
2. JWT Filter는 발급된 Access Token을 인증 헤더인 Bearer에 담아 validateAccessToken 메서드에서 Access Token 검증을 받게 된다.
   1. AccessToken이 만약 null이라면 발급도 받지 않은 경우이므로 다음 필터로 넘어가게 된다.
   2. AccessToken이 만료됐다면 서버는 ProcessRefreshToken 메서드를 호출해서 RefreshToken 검증, replayAttack 감지, Access & Refresh 재생성 처리를 진행하게 된다.
3. Access Token이 문제가 없는 경우라면 setSecurityContextWithUserDetails 메서드로 진입해서 Access Token의 Payload에 들어있는 사용자 정보를 ContextHolder에 저장해서 인증/인가 처리한다.

## 코드
### JWTFilter
```
public class JWTFilter extends OncePerRequestFilter {
    private final JWTUtil jwtUtil;
    private final ReissueService reissueService;

    public JWTFilter(JWTUtil jwtUtil, ReissueService reissueService) {
        this.jwtUtil = jwtUtil;
        this.reissueService = reissueService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        StringBuilder accessToken = new StringBuilder();
        if (validateAccessToken(accessToken, request, response, filterChain)) {
            setSecurityContextWithUserDetails(accessToken.toString());
            filterChain.doFilter(request, response);
        }
    }

    private boolean validateAccessToken(StringBuilder accessToken, HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 헤더에서 access 토큰을 꺼냄
        String authorization = request.getHeader("Authorization");
        if (authorization == null || !authorization.startsWith("Bearer ")) {
            System.out.println("token null");
            filterChain.doFilter(request, response);
            return false;
        }

        accessToken.append(authorization.split(" ")[1]);
        if (accessToken.isEmpty()) {
            System.out.println("accessToken null");
            filterChain.doFilter(request, response);
            return false;
        }

        // Access Token이 만료됐다면 서버는 Refresh Token을 Client에게 요청한다.
        boolean isAccessTokenExpired = false;
        try {
            jwtUtil.isExpired(accessToken.toString());
        } catch (ExpiredJwtException e) {
            isAccessTokenExpired = true;
            boolean isRefreshTokenValid = reissueService.processRefreshToken(request, response);
            // Refresh Token도 만료됐다면 재로그인을 요청한다.
            if (!isRefreshTokenValid) {
                System.out.println("다시 로그인해야 합니다.");
            }

            return false;
        }

        // 토큰이 access인지 확인 (발급시 페이로드에 명시)
        if (!jwtUtil.getCategory(accessToken.toString()).equals("access")) {
            PrintWriter writer = response.getWriter();
            writer.print("invalid access token");
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }

        return true;
    }

    private void setSecurityContextWithUserDetails(final String accessToken) {
        String username = jwtUtil.getUsername(accessToken);
        String role = jwtUtil.getRole(accessToken);

        Member userEntity = new Member();
        userEntity.setUsername(username);
        userEntity.setRole(role);
        CustomUserDetails customUserDetails = new CustomUserDetails(userEntity);

        Authentication authToken = new UsernamePasswordAuthenticationToken(customUserDetails, null, customUserDetails.getAuthorities());
        SecurityContextHolder.getContext().setAuthentication(authToken);
    }
}
```

Access Token 검증을 하는 validateAccessToken을 통과하지 못했다면 SecurityContext에 사용자 정보를 넣을 수 없게 했다. 그 이유는 AccessToken의 Payload에서 사용자 정보를 가져오기 때문이다.  

validateAccessToken 내부에 AccessToken이 만료된 경우에만 RefreshToken을 사용해서 재발급 요청을 서버에게 보낸다. 왜냐하면 null일 경우는 애초에 Login이 되지 않은 상황이라 발급도 받은 적이 없을테고, 그리고 토큰이 access인지 확인을 그 뒤에 하는 이유는 만료 이후에는 토큰에 접근할 수 없기 때문이다.  

### ReissueService
```
@Service
@RequiredArgsConstructor
public class ReissueService {
    private final RefreshRepository refreshRepository;
    private final CookieUtils cookieUtils;
    private final JWTUtil jwtUtil;
    private final RedisUtil redisUtil;

    // RefreshToken을 재발급하거나, 블랙리스트에 추가한다.
    public boolean processRefreshToken(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        final String refreshToken = findRefreshToken(request, response);
        if (!validateRefreshToken(refreshToken, response)) {
            return false;
        }

        if (rejectReplayAttack(refreshToken, response)) {
            return false;
        }

        // Access Token이 만료됐고, 그리고 Refresh Token이 유효하므로 Refresh Token Rotation을 호출한다.
        generateAccessAndRefreshTokens(refreshToken, response);
        return true;
    }

    public String findRefreshToken(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String refreshToken = null;
        refreshToken = cookieUtils.findCookie("Refresh", request.getCookies());
        if (refreshToken == null) {
            PrintWriter writer = response.getWriter();
            writer.print("refresh token null");
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return null;
        }

        return refreshToken;
    }

    public boolean validateRefreshToken(String refreshToken, HttpServletResponse response) throws ServletException, IOException {
        try {
            jwtUtil.isExpired(refreshToken);
        } catch (ExpiredJwtException e) {
            PrintWriter writer = response.getWriter();
            writer.print("refresh token expired");
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return false;
        }

        // 토큰이 refresh인지 확인 (발급시 페이로드에 명시)
        if (!jwtUtil.getCategory(refreshToken).equals("refresh")) {
            PrintWriter writer = response.getWriter();
            writer.print("invalid refresh token");
            response.setStatus(HttpServletResponse.SC_BAD_REQUEST);
            return false;
        }

        return true;
    }

    private boolean rejectReplayAttack(final String refreshToken, HttpServletResponse response) {
        // 블랙리스트에 추가된 Refresh Token인지 확인한다.
        String refreshTokenTest = "eyJhbGciOiJIUzI1NiJ9.eyJjYXRlZ29yeSI6InJlZnJlc2giLCJ1c2VybmFtZSI6ImFkbWluIiwicm9sZSI6IlJPTEVfQURNSU4iLCJpYXQiOjE3NDM5MTQ3MTEsImV4cCI6MTc0MzkxNTMxMX0.lcEdN0UfrzUdE5KkvKY3bGM0N1Wdcx8Hbo1W_d1nLtA";
        if (redisUtil.hasKeyBlacklistEntries(refreshTokenTest)) {
            // Replay Attack 감지
            // 두 가지 경우로 나뉜다.
            // 1. 클라이언트가 재발급을 마친 상태에서, 이미 이전에 해커가 클라이언트의 토큰을 탈취한 다음 재발급을 신청한 경우
            // 2. 해커가 클라이언트의 토큰을 탈취 후에 재발급을 마친 상태에서 클라이언트가 재발급을 신청한 경우
            // 서버는 재발급 대상자를 식별할 수 없으므로, 이미 재발급된 토큰은 블랙리스트에 추가하고, 현재 재발급은 무효 처리한다.
            final String userInfo = redisUtil.getBlacklistEntries(refreshTokenTest);
            if (redisUtil.hasKey(userInfo)) {
                final String issuedRefreshToken = redisUtil.get(userInfo);
                if (issuedRefreshToken != null) {
                    redisUtil.delete(userInfo);
                    redisUtil.setBlacklistEntries(issuedRefreshToken, userInfo, 300000L);
                }
            }

            System.out.println("Replay Attack 감지");
            response.addCookie(cookieUtils.createCookie("Refresh", null, 600000));

            return true;
        }

        return false;
    }

    private void generateAccessAndRefreshTokens(String refreshToken, HttpServletResponse response) {
        final String username = jwtUtil.getUsername(refreshToken);
        final String role = jwtUtil.getRole(refreshToken);

        // 이전 Refresh Token을 Redis에서 제거, Blacklist에 추가한다.
        redisUtil.delete(username);
        redisUtil.setBlacklistEntries(refreshToken, username, 300000L);

        String newAccess = jwtUtil.createJwt("access", username, role, jwtUtil.getAccessExpiry());
        String newRefresh = jwtUtil.createJwt("refresh", username, role, jwtUtil.getRefreshExpiry());

        // 새로 생성된 Refresh Token은 Redis에서 관리한다.
        redisUtil.set(username, newRefresh, 300000L);

        response.setHeader("Authorization", "Bearer " + newAccess);
        response.addCookie(cookieUtils.createCookie("Refresh", newRefresh, 24 * 60 * 60));
    }

    public boolean existsByRefresh(String refresh) {
        return refreshRepository.existsByRefresh(refresh);
    }

    public void deleteByRefresh(String refresh) {
        refreshRepository.deleteByRefresh(refresh);
    }

    public void addRefreshEntity(String username, String refresh, Long expiredMs) {
        Date date = new Date(System.currentTimeMillis() + expiredMs);

        RefreshEntity refreshEntity = new RefreshEntity();
        refreshEntity.setUsername(username);
        refreshEntity.setRefresh(refresh);
        refreshEntity.setExpiration(date.toString());

        refreshRepository.save(refreshEntity);
    }
}
```

processRefreshToken는 RefreshToken을 검증해서 유효하다면 Access Token & Refresh Token을 새로 발급과 동시에 기존 Refresh Token은 Blacklist에 추가해서 ReplayAttack 공격을 대비한다.  

processRefreshToken 내부에는 크게 3단계로 구성되어 있다. 
1. validateRefreshToken
2. rejectReplayAttack
3. generateAccessAndRefreshTokens

generateAccessAndRefreshTokens을 제외한 단계에서 RefreshToken이 유효하지 않거나, replayAttack 공격이 들어왔다면 재발급을 하지 못하도록 처리하였다.  

### rejectReplayAttack
blacklist에 추가된 Refresh Token인지 확인해서, 맞다면 접근하지 못하도록 막는 역할을 수행한다.  

replay attack에 감지되는 상황은 두 가지로 나뉜다.
1. 클라이언트가 재발급을 마친 상태에서, 이미 이전에 해커가 클라이언트의 토큰을 탈취한 다음 재발급을 신청한 경우
2. 해커가 클라이언트의 토큰을 탈취 후에 재발급을 마친 상태에서 클라이언트가 재발급을 신청한 경우  

서버는 재발급 대상자를 식별할 수 없으므로, 이미 재발급된 토큰은 블랙리스트에 추가하고, 현재 재발급은 무효 처리한다.

### generateAccessAndRefreshTokens
두 단계를 거쳐서 Access Token & Refresh Token을 재발급하는 역할을 수행한다. 이미 눈치챘겠지만 (RTR)Refresh Token Rotation이다.  

이 단계에서는 현재 소유한 Refresh Token이 저장된 Redis에서 제거하고 blacklist에 옮기는 작업을 수행한다. 그 이유는 여전히 replay attack을 대비하기 위해서다. 그리고 새로 발급된 Refresh Token 역시 Redis에서 저장한다. 왜냐하면 추후에 해커가 탈취한 token을 가지고 인증을 시도할 때, redis에서 관리하고 있는 token과 다르다는 것을 대조하기 위함이다.

