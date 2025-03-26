---
title: 5 - Spring Security에 JWT 도입
categories: ci-cd-auth-project
---

# 코드 도입
이전 시간에 학습한 Security 설계도를 바탕으로 JWT 시스템을 도입하려고 한다. 되도록이면 코드가 설계도에서 어느 곳에 위치하고 있고, 그리고 무슨 역할을 왜? 이렇게 설계되어 있는지 등 의문 중심으로 이해한 대로 정리하려고 한다.  

## UsernamePasswordAuthenticationFilter
```
public class LoginFilter extends UsernamePasswordAuthenticationFilter {

    private final AuthenticationManager authenticationManager;
    private final JWTUtil jwtUtil;

    public LoginFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil) {

        this.authenticationManager = authenticationManager;
        this.jwtUtil = jwtUtil;
    }

    @Override
    public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {

        String username = obtainUsername(request);
        String password = obtainPassword(request);

        UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(username, password, null);

        return authenticationManager.authenticate(authToken);
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authentication) {

        CustomUserDetails customUserDetails = (CustomUserDetails) authentication.getPrincipal();

        String username = customUserDetails.getUsername();

        Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
        Iterator<? extends GrantedAuthority> iterator = authorities.iterator();
        GrantedAuthority auth = iterator.next();

        String role = auth.getAuthority();

        String token = jwtUtil.createJwt(username, role, 60*60*10L);

        response.addHeader("Authorization", "Bearer " + token);
    }

    @Override
    protected void unsuccessfulAuthentication(HttpServletRequest request, HttpServletResponse response, AuthenticationException failed) {

        response.setStatus(401);
    }
}
```

### UsernamePasswordAuthenticationFilter는 Security Filter
사용자 이름과 비밀번호 사용 시 사용자의 자격 증명을 인증하는 기본 필터로 사용된다.  

일반적으로 사용하는 Form Login 방식을 Custom 하여 동작을 시킬 것이기 때문에 Form Login 방식에서 사용하는 UsernamePasswordAuthenticationFilter를 사용한다.  

이 필터는 Security Filter이기 때문에 브라우저 요청이 들어오면 컨트롤러 이전에 먼저 호출이 된다. 그 이유는 ServletFilter의 구현체 DelegatingFilterProxy가 Spring의 ApplicationContext에서 Bean으로 위임 후 FilterChainProxy가 이를 제어했기 때문이다.  

### FilterChainProxy에 적용 방법
```
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .csrf((auth) -> auth.disable());

        http
                .formLogin((auth) -> auth.disable());

        http
                .httpBasic((auth) -> auth.disable());

        http
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/login", "/", "/join").permitAll()
                        .anyRequest().authenticated());
        http
                .addFilterAt(new LoginFilter(authenticationManager(authenticationConfiguration)), UsernamePasswordAuthenticationFilter.class);
}
```

### UsernamePasswordAuthentication의 부모는 AbstractAuthenticationProcessingFilter
UsernamePasswordAuthenticationFilter는 AbstractAuthenticationProcessingFilter의 구현체이다. AbstractAuthenticationProcessingFilter는 사용자의 자격 증명을 인증하는 기본 필터인데, 클라이언트가 자격 증명 요청을 하게 되면 AbstractAuthenticationProcessingFilter는 HttpServletRequest에서 인증할 Authentication 객체를 생성한다. 생성되는 Authentication 객체의 타입은 AbstractAuthenticationProcessingFilter의 하위 클래스에 따라 다르다. **바로 여기서 Form 방식을 사용할 것이기 때문에 UsernamePasswordAuthenticationFilter를 구현하여 Security Filter에 적용하였다.**  

### AbstractAuthenticationProcessingFilter의 동작 과정
1. 자격 증명 제출 시
   * UsernamePasswordAuthenticationFilter는 HttpServletRequest에 제출된 사용자 이름과 비밀번호를 사용해 UsernamePasswordAuthenticationToken을 생성
2. AuthenticationManager에 전달
   * 생성된 Authentication 객체는 인증을 위해 AuthenticationManager에 전달
3. 인증 실패 시
   * SecurityContextHolder 초기화
   * AuthenticationFailureHandler가 호출
4. 인증 성공 시
   * 인증된 Authentication 객체는 SecurityContextHolder에 저장
   * AuthenticationSuccessHandler가 호출  

<br>

## UserDetailsService & UserDetails
```
@Service
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    public CustomUserDetailsService(UserRepository userRepository) {

        this.userRepository = userRepository;
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
				
				//DB에서 조회
        UserEntity userData = userRepository.findByUsername(username);

        if (userData != null) {
						
						//UserDetails에 담아서 return하면 AutneticationManager가 검증 함
            return new CustomUserDetails(userData);
        }

        return null;
    }
}
```
```
public class CustomUserDetails implements UserDetails {

    private final UserEntity userEntity;

    public CustomUserDetails(UserEntity userEntity) {

        this.userEntity = userEntity;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {

        Collection<GrantedAuthority> collection = new ArrayList<>();

        collection.add(new GrantedAuthority() {

            @Override
            public String getAuthority() {

                return userEntity.getRole();
            }
        });

        return collection;
    }

    @Override
    public String getPassword() {

        return userEntity.getPassword();
    }

    @Override
    public String getUsername() {

        return userEntity.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {

        return true;
    }

    @Override
    public boolean isAccountNonLocked() {

        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {

        return true;
    }

    @Override
    public boolean isEnabled() {

        return true;
    }
}
```

UserDetailsService는 사용자가 로그인할 때 특정 사용자에 대한 세부 정보를 조회하고 반환하게 되는데, 이 반환 객체가 UserDetail이다.  

### 왜 Customize를 했는가?
기본적으로 UserDetailsService를 제공해 주지만, 프로젝트 요구 조건에 따라 변경해야 한다.
1. DB 조회 & 저장
Entity의 정보를 DB에 저장, 불러와야 하므로 DAO를 이용하여 user 정보를 받아와야 한다.
2. Entity의 인증과 권한
원하는 객체의 인증과 권한 체크가 필요하기 때문이다.  

<br>

## JWTUtil 12.3
```
@Component
public class JWTUtil {

    private SecretKey secretKey;

    public JWTUtil(@Value("${spring.jwt.secret}")String secret) {

        // secret 값을 바이트 배열로 변환한 후, HS256 알고리즘으로 서명할 SecretKey 객체를 생성
        secretKey = new SecretKeySpec(secret.getBytes(StandardCharsets.UTF_8), Jwts.SIG.HS256.key().build().getAlgorithm());
    }

    public String getUsername(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("username", String.class);
    }

    public String getRole(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token).getPayload().get("role", String.class);
    }

    public Boolean isExpired(String token) {

        return Jwts.parser().verifyWith(secretKey).build().parseSignedClaims(token). // JWT에서 서명된 클레임을 파싱
                getPayload().getExpiration().
                before(new Date()); // 현재 시간이 만료일보다 이후인지 확인
    }

    public String createJwt(String username, String role, Long expiredMs) {

        return Jwts.builder()
                .claim("username", username)
                .claim("role", role)
                .issuedAt(new Date(System.currentTimeMillis())) // 발행일을 현재 시간으로 설정
                .expiration(new Date(System.currentTimeMillis() + expiredMs)) // 만료일을 현재 시간 + expiredMs로 설정
                .signWith(secretKey)
                .compact(); // JWT 문자열로 변환하여 반환
    }
}
```

JWTUtil은 주로 JWT를 생성, 검증 및 처리하는 데 사용되는 유틸리티 클래스이다. JWTUtil을 사용해서 사용자의 Token을 관리하려고 한다.

### 라이브러리 설치
- [https://github.com/jwtk/jjwt#installation](https://github.com/jwtk/jjwt#installation)  

참고로 현재 버전은 jwtUtil 12.3 버전을 사용 중이지만, 위 공식문서 사이트는 최신 12.6 버전의 기준으로 설명을 한다. jwtutil에 대한 라이브러리 설치, 메서드 사용 방법에 대해 자세히 나와있다. 









