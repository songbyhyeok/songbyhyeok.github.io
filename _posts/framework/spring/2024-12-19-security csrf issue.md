---
title: Spring Security, Thymeleaf CSRF Issue
categories: spring
---

# 회원가입 로직 구현중... [Intro]
회원가입 로직 개발을 하고 있었고, 다음과 같은 기술 스택들을 사용하게 되었다.  
**Spring Security, Thymeleaf(SSR), MVC, Html Form tag, Axios**  
상황은 순조롭게 개발이 진행되고 있었지만, 휴대폰 인증 비동기 요청으로 'post' 방식 사용하였을 때 이슈가 발생하였다.  

**Http Method**  
Get 외에 Post, Put, Delete 메소드들은 Http 상태 변경 요청 즉, 서버에게 사용자 권한을 악용한 DBMS 침투가 가능하다.  

## 휴대폰 인증 비동기 요청 상황
![image](https://github.com/user-attachments/assets/88bdd35f-539e-4717-af91-266e887cde78)  
두 가지 버튼은 공통적으로 비동기 통신 방식인 'Post' 로 요청을 해야했고, 비동기 통신 라이브러리 'axios'를 채택, 이유는 'fetch' 방식과 비교해서 좀 더 직관적이면서 json 변환이 더 간편하기 때문이다.  

![image](https://github.com/user-attachments/assets/3c57de3f-6025-4cfa-95e6-664b99eaa1a8)  
![image](https://github.com/user-attachments/assets/aedf7ade-ab9d-4a8a-b661-f29bde0944a3)  

'Post' 요청을 보내자, 403 에러로 응답하였다. **왜? 403 에러 메시지를 전달하였을까?** 

## 403 Forbidden [Issue]
**작동중인 서버에 클라이언트의 요청이 도달했으나, 서버가 클라이언트의 접근을 거부할 때 반환하는 HTTP 응답 코드이자 오류 코드다.**  
**"- 나무위키 -"**  

이제는 문제로 정의해야 한다. 서버는 왜? 제어 권한을 인증된 사용자에게 부여하지 않는 이유가 무엇일까?  
<br>

# CSRF 보안 설정 [Problem]
Security는 기본적으로 **CSRF 보안 설정**이 되어 있다. 그렇다는 것은 간략하게 CSRF 보안 목적을 파악해야 할 것이다.  

## CSRF Protection 목적 이해
```
    Cross-Site Request Forgery (CSRF or XSRF) is a type of security vulnerability in web applications. 
    Without protection from CSRF, a Jenkins user or administrator visiting 
    some other web site would allow the operator of that site to perform actions in Jenkins as the victim.    
    ....CSRF로부터 보호되지 않으면 다른 웹 사이트를 방문하는 사용자, 관리자가 공격자 의도대로 서버 공격
    // 출처 https://www.jenkins.io/doc/book/security/csrf-protection/
```

Security는 현재 요청된 사용자가 **'공격자'**인지 식별할 수 없어. 접근 권한을 부여하지 않는다.  
즉, 내가 **'사용자'** 라는 사실을 증명해야 한다.  

## CSRF Token
Security는 CSRF Token이라는 난수 값을 만들어서 사용자에게 제공 후 이를 인증 과정을 통해 제어 권한을 결정하게 된다.  
그러니까, **'증명 도구'**로 쓰인다는 것이다. 이제는 **'증명 방법'**들을 모색할 차례이다.  
<br>

# CSRF 방어 방법 [Solutions]
공격에 대한 방어는 클라이언트와 서버에서 각각 다른 방식으로 처리된다.  

## 서버 토큰 관리
```
@Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf
                                .csrfTokenRepository()
                )
        return http.build();
    }
```

토큰을 어떻게 관리할지에 대한 방법을 다룬다. 즉, 서버는 CSRF 공격을 방어하기 위해 적절한 방식으로 토큰을 생성하고, 이를 검증하는 방법을 설정한다.  
클라이언트의 요청에 포함되는 토큰을 서버는 저장된 토큰 값과 일치하는지 확인해 요청을 검증하는 방식으로 사용자를 구별한다. csrf 토큰은 CsrfConfigurer의 csrfTokenRepository에서 저장 및 검증되며, 방식은 **Cookie, Session, Lazy의 Repository**가 있다.  

### Cookie
```
@Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf
                        .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                )

    private CookieCsrfTokenRepository cookieCsrfRepository() {
        CookieCsrfTokenRepository csrfRepository = new CookieCsrfTokenRepository();

        csrfRepository.setCookieHttpOnly(false);
        csrfRepository.setHeaderName("X-CSRF-TOKEN");
        csrfRepository.setParameterName("_csrf");
        csrfRepository.setCookieName("XSRF-TOKEN");
        //csrfRepository.setCookiePath("..."); // 기본값: request.getContextPath()

        return csrfRepository;
    }
    }
```

**CookieCsrfTokenRepository**  
CSRF 토큰을 클라이언트 브라우저 쿠키에 저장한다. 기본 주입은 new CookieServerCsrfTokenRepository()이며, 클라이언트 측 스크립트에서 쿠키 제어 허용이 필요하다면 static.withHttpOnlyFalse()를 추가하면 된다. 다만, 해당 옵션은 공격자가 자신의 쿠키를 사용자 브라우저에 심어 XSS 공격 가능성을 내포한다. 좀 더, 구체적인 쿠키의 토큰 설정이 필요하면 해당 repository를 custom 구현하면 된다.  
  
쿠키 토큰 관리 방식은 위에 언급했듯, 위험성을 내포하고 있어서 아래에서 소개할 세션 저장 방식을 사용하는 것을 좋다.  
  
<img src="https://github.com/user-attachments/assets/52ded04f-7b1a-45fd-9cb5-f2f303ffdf77" style="width: 60%; height: auto">  
CSRF Token이 브라우저에 저장된 것을 확인할 수 있다.  

### Session
```
@Configuration
@EnableWebSecurity
public class SecurityConfig{
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf
                                .csrfTokenRepository(sessionCsrfRepository())
                        //.csrfTokenRepository(new HttpSessionCsrfTokenRepository())
                )
                .authorizeHttpRequests((auth) -> auth
                        .requestMatchers("/css/**", "/js/**").permitAll()
                        .requestMatchers("/", "/login", "/signup/**").permitAll()
                        .anyRequest().authenticated()
                );

        return http.build();
    }

    private HttpSessionCsrfTokenRepository sessionCsrfRepository() {
        HttpSessionCsrfTokenRepository csrfRepository = new HttpSessionCsrfTokenRepository();

        // HTTP 헤더에서 토큰을 인덱싱하는 문자열 설정
        csrfRepository.setHeaderName("X-CSRF-TOKEN");
        // URL 파라미터에서 토큰에 대응되는 변수 설정
        csrfRepository.setParameterName("_csrf");
        // 세션에서 토큰을 인덱싱 하는 문자열을 설정. 기본값이 무척 길어서 오버라이딩 하는 게 좋아요.
        // 기본값: "org.springframework.security.web.csrf.HttpSessionCsrfTokenRepository.CSRF_TOKEN"
        csrfRepository.setSessionAttributeName("CSRF_TOKEN");

        return csrfRepository;
    }
}
```

**HttpSessionCsrfTokenRepository**  
저장 방식의 기본값이며, CSRF 토큰을 서버의 HttpSession에 저장한다. 기본 주입은 new HttpSessionCsrfTokenRepository()이고, session의 token 관리가 필요하면 해당 repository를 custom 구현하면 된다.  
                        
### Lazy
LazyCsrfTokenRepository, 데코레이터 패턴으로 토큰을 느긋하게 저장한다.

### CSRF.disable()
```
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf
                        .disable()  // CSRF 비활성화
                )
}
```

csrf 보호를 비활성화 시킨다. 해당 옵션 사용 목적은 환경에 따라 달라진다.  

**CSRF Referer**  
Rest API 통신 환경에서는 stateless 특성에 따라 Oauth, JWT 방식을 사용하기 때문에 CSRF 공격으로부터 안전하다. 따라서 csrf 설정을 disable 하는 것이 일반적이다. 그렇지만 JWT 토큰 사용 시 쿠키에 저장할 경우 공격 가능성이 생기기 때문에 이때는, 활성화 후 Restful 환경 특성상 CSRF 토큰을 발급할 VIEW 페이지와 같은 로직이 없기 때문에 토큰 방식이 아닌 Referer 방식을 사용한다. 이 방식은 HTTP Referer 헤더를 통해 요청의 출발점, 이전 URL등을 검증한다.  

**로컬 개발**  
클라이언트에서만 개발하는 경우 비활성화하는 것을 권장하고 있다.

## 토큰 요청 및 생성 과정
어떤 요청을 보낼 때 해당 토큰을 어떻게 포함시킬지에 대해 다룬다. 즉, 클라이언트는 서버가 요구하는 토큰을 요청 시점에 정확히 전달할 수 있도록 처리한다.

### 서버에서 클라이언트로 CSRF 토큰 발급
기본 동작은 SSR 세션 방식으로 설정되어 있으며, 웹 동기/비동기 통신 방식에 따라 저장 방식이 상이하다.

**동기화된 패턴 방식의 CSRF**  
```
<div class="row mb-3 justify-content-center mt-4">
    <div class="col-sm-2 d-flex justify-content-center">
        <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
        <button type="submit" class="btn btn-secondary w-100">확인</button>
    </div>
</div>
```

Thymeleaf SSR 방식에서 쓰이는, 폼 제출의 숨겨진 필드에 csrf 토큰을 동봉해서 요청 및 인증하는 방법이다. CSRF 토큰은 동기화된 패턴의 쿠키로 전송되거나, 서버 로그나 URL에서 노출되어서는 안된다. 이 방법은 말그대로 간단한 동기 통신할 경우 쓰이는 방법이지만, 사용자 지정 헤더가 있는 요청은 자동으로 동일 출처 정책에 따라 달라지므로 JS를 통해 사용자 지정 HTTP 요청 헤더에 CSRF 토큰을 삽입하는 비동기 통신 전달 방법을 사용하는 것이 더 안전하다.  

**비동기 패턴 방식의 CSRF**  
```
html
<head>
    <meta name = "_csrf" th:content = "${_csrf.token}" />
        <!-- 기본 헤더 이름은 X-CSRF-TOKEN입니다 -->
    <meta name = "_csrf_header" th:content = "${_csrf.headerName}" />
</head>
js
//html meta에서 csrf token 가져오기
const csrfHeader = document.querySelector('meta[name="_csrf_header"]').content;
const csrfToken = document.querySelector('meta[name="_csrf"]').content;
const jsonHeaders = {
    'Content-Type': 'application/json',
    [csrfHeader]: csrfToken,
};
switch (method.toUpperCase()) {
    case 'POST':
    response = await axios.post(url, res, {headers: jsonHeaders});
    break;
}
```

이 방법은 비동기 통신 시 사용하는 방법이다. 비동기 JSON 포멧 방식은 HTTP 매개변수 내에서 CSRF 토큰을 제출할 수 없다. 대신에 HTTP 헤더 내에서 토큰을 제출할 수는 있기 때문에, html head 태그 내부 meta 태그에 csrf 데이터를 저장하고, 메타 태그에 CSRF 토큰이 포함되면 JavaScript 코드는 메타 태그를 읽고 CSRF 토큰을 헤더로 포함할 수 있다. 비동기 통신 전달 방법은 동기 통신 방법보다 효력이 있어, 해당 방법을 사용하는 것이 좋다.  

## 요청에 대한 응답값 전달
### 세션
```
@RequestMapping("/csrf")
@Controller
public class CsrfController {

    private static final Logger logger = LoggerFactory.getLogger(CsrfController.class);

    @RequestMapping(method = RequestMethod.GET)
    public ResponseEntity<String> getOrCreateCsrfToken(HttpSession session, HttpServletRequest request) {
        final DefaultCsrfToken csrfToken = (DefaultCsrfToken) session.getAttribute(
                "CSRF_TOKEN");

        assert(csrfToken == request.getAttribute(csrfToken.getParameterName()));

        return ResponseEntity.ok()
                .header(csrfToken.getHeaderName(), csrfToken.getToken()).body("Check your response header!");
    }
}
```

session 방식의 토큰 값을 주입 후, 클라이언트가 요청을 보낼 경우 csrfFilter가 끼어들어 csrf 토큰 생성 및 HttpSession과 HttpServletRequest에 저장한다. 그리고 이를 컨트롤러 단에서 csrf 토큰을 보내줄 수 있다.

### 쿠키
cookie 방식의 토큰 값을 주입 후, 클라이언트가 요청을 보낼 경우 csrfFilter가 끼어들어 csrf 토큰 생성 및 HttpServletRequest에 저장한다. 최초에 한하여 응답 헤더에 Set-Cookie로 CSRF 토큰을 내려 보낸다. 최초 접근 이후 클라이언트는 CSRF 토큰을 쿠키로 보유하는 상태로, 요청 헤더 혹은 파라미터에 동일 토큰을 서버로 실어 보낸다. 서버는 쿠키와 요청 파라미터 또는 헤더를 비교하여 유효한 CSRF가 담겼는지 검증하게 된다.  
<br>

# Security CSRF 구조 및 동작원리 [Structure]

## CSRF Methods
**csrfTokenRepository**  
CSRF 토큰의 생성, 저장에 사용되는 인터페이스. 해당 클래스를 구현한 클래스에게 위임 시킨다.  
**csrfTokenRequestHandler**  
요청 헤더, 파라미터의 CSRF 토큰을 제출할 경우 이를 검증하는 역할로서, 추가 작업을 수행하거나 토큰 커스텀 가능.  
**ignoringRequestMatchers**  
CSRF 관리가 필요하지 않은 URL을 기입(회원가입 등 특정 페이지들은 피해를 줄 수 없다고 판단될 경우 사용) 

## CsrfFilter
DefaultSecurityFilterChain에 기본적으로 등록되는 필터로 여섯 번째에 위치해 있다.  
이 필터는 CSRF 공격을 방지하기 위한 역할로서, HTTP 메소드 중 GET, HEAD, TRACE, OPTIONS 메소드를 제외한 요청에 대해서 토큰 검증을 진행하는데,  
요청 시 토큰을 서버 저장소에 저장 후 클라이언트에게도 전송하며, 그 후 해당하는 요청에 대해서 서버에 저장된 토큰과 비교 검증을 진행한다.  
참고로 매 요청마다 토큰은 난수 값으로 생성된다.  

### 요청 흐름
<img src="https://github.com/user-attachments/assets/d9d84dfc-031b-4f2f-a5a9-c51f0ec9510f" style="width: 70%; height: auto">  
<img src="https://github.com/user-attachments/assets/6cfb183d-c754-4815-96cd-cece654edc64" style="width: 70%; height: auto">  
1번에서 지연 로딩을 통해 토큰을 가져오고, 3번에서 요청이 CSRF 보호가 필요한지 조건문 처리가 발동한다.  
이때, 필요하다고 판단될 경우 4번 로직 이후 6번에서 토큰을 검증한다.  
여기서 또 조건 처리에 따라 토큰이 다르면 403 Forbidden 발생하고, 그게 아니고 토큰이 일치한다면 다음 필터를 진행하게 된다.  

### 코드 관점
```
http
        .csrf((csrf) -> csrf.disable());
```

http.csrf() 구문으로 동작하며, 만약 csrf filter의 동작을 해제하고 싶다면 .disabled()를 호출하면 된다. 그리고 커스텀 SecurityFilterChain을 생성해도 등록된다.  

**CsrfFilter 클래스**  
```
public final class CsrfFilter extends OncePerRequestFilter {

}
```

CsrfFilter는 추상클래스OncePerRequestFilter의 구현체로, 요청 당 단 한번만 필터 로직을 수행하는 필터이다.  

**주요 로직 : doFilterInternal**  
```
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
		throws ServletException, IOException {
		
	// 토큰을 토큰 저장소로 부터 불러옴
	DeferredCsrfToken deferredCsrfToken = this.tokenRepository.loadDeferredToken(request, response);
	// 다음 요청을 위해 request에 추가
	request.setAttribute(DeferredCsrfToken.class.getName(), deferredCsrfToken);
	this.requestHandler.handle(request, response, deferredCsrfToken::get);
	
	// HTTP 메소드 확인 후 CSRF 검증이 필요 없는 메소드면 다음 필터로 넘김
	if (!this.requireCsrfProtectionMatcher.matches(request)) {
		if (this.logger.isTraceEnabled()) {
			this.logger.trace("Did not protect against CSRF since request did not match "
					+ this.requireCsrfProtectionMatcher);
		}
		filterChain.doFilter(request, response);
		return;
	}
	
	// 서버 저장 토큰
	CsrfToken csrfToken = deferredCsrfToken.get();
	// 클라이언트에서 온 토큰
	String actualToken = this.requestHandler.resolveCsrfTokenValue(request, csrfToken);
	
	// 클라이언트로 부터 온 토큰과 서버 저장소의 토큰을 비교 검증
	if (!equalsConstantTime(csrfToken.getToken(), actualToken)) {
		boolean missingToken = deferredCsrfToken.isGenerated();
		this.logger
			.debug(LogMessage.of(() -> "Invalid CSRF token found for " + UrlUtils.buildFullRequestUrl(request)));
		AccessDeniedException exception = (!missingToken) ? new InvalidCsrfTokenException(csrfToken, actualToken)
				: new MissingCsrfTokenException(actualToken);
		this.accessDeniedHandler.handle(request, response, exception);
		return;
	}
	
	// 다음 필터로 넘김
	filterChain.doFilter(request, response);
}
```

<br>  

# 그래서 무슨 방법을 사용했는가? [Solution]
## 서버 저장 방식 채택
서버 Repository의 방식인 Cookie, Session, Lazy 중 Session을 채택하였다. 그 이유는 우선 쿠키는 브라우저에 토큰을 저장하는 방식이라 구현 및 검증 과정이 상대적으로 쉽지만, 브라우저의 토큰에 접근해서 XSS 공격을 할 수 있기 때문에 사용하지 않았고, Lazy는 솔직하게 잘모르겠어서 넘어갔다. 그리고 저장 방식이 아니지만 disable() 를 사용하지 않는 이유는 적합한 RESTFul 환경이 아닌 점과 jwt 토큰으로 저장하지 않기 때문에, 현재 사용 중인 SSR 방식의 Thymeleaf와 MVC를 사용한 환경은 Session Token 방식으로 인증하는 것이 가장 적합하다고 판단하였다.  

## 클라이언트 요청 방식
Meta 태그에 csrf를 발급하여 method 'GET'을 제외한 모든 요청을 header에 넣어 csrf 문제를 해결하였다.  
<br>

# 결론 [Conclusion]
휴대폰 인증 기능 구현 도중 403 에러를 맞닥뜨리고, CSRF 보안이 문제가 된다는 것을 파악하였다. 그리고 이를 해결하기 위한 사용자 인증 도구는 CSRF Token이었고, 인증 방법은 서버에서 토큰 저장 관리 방법과 클라이언트에서는 동기 방식에 따라 요청 시 토큰을 어떻게 포함시켜야 할지에 대해 다루어보았다. 전체 기능 구현 중에 CSRF란 ISSUE는 정말 보잘 것 없고 왜 이런 용어도 생소한 걸 넣어놨지란 의문이 들었지만, 실상은 정말 중요한 보안 기술이고 이를 개발자가 적합한 방법을 채택해서 토큰 제어 및 관리를 할 수 있다는 점에서 나름 의미가 있었고, Security 구조 및 동작원리에 있어 내용이 방대하지만, 그 중에 일부인 CSRF 필터 구조 및 동작 원리에 대해 조금이나마 이해할 수 있는 시간이어서 많은 공부가 되었다고 생각한다.  
<br>

# 참고 [References]
* [크로스 사이트 요청 위조(CSRF)](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)
* [[스프링 Security] CSRF 토큰 이야기 - 그래서 개발자는 뭘 하면 되죠](https://binchoo.tistory.com/46)
* [[개발] SPA vs. MPA: 단일 페이지 앱과 다중 페이지 앱 비교](https://dailybit.co.kr/entry/%EA%B0%9C%EB%B0%9C-SPA-vs-MPA-%EB%8B%A8%EC%9D%BC-%ED%8E%98%EC%9D%B4%EC%A7%80-%EC%95%B1%EA%B3%BC-%EB%8B%A4%EC%A4%91-%ED%8E%98%EC%9D%B4%EC%A7%80-%EC%95%B1-%EB%B9%84%EA%B5%90)
* [SPA(Single Page Application), MPA(Multi Page Application)](https://cjwoov.tistory.com/97)
* [CSRF 공격을 막기 위한 CSRF 토큰](https://qkrqkrrlrl.tistory.com/170)
* [Spring Security - 크로스 사이트 요청 위조(CSRF)](https://leeheeweon.github.io/2023/11/16/spring-security-csrf/)
* [14. CsrfFilter](https://www.devyummi.com/page?id=669a5f7a94f57583e6bb35c9)