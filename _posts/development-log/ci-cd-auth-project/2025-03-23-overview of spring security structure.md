---
title: 4 - Spring Security 구조, 흐름
categories: ci-cd-auth-project
---

# JWT Token을 사용하기 앞서
JWT 동작 과정을 이해하기 위해서는 ServletFilterChain과 SpringSecurity 구조와 동작 원리에 대해서 학습할 필요가 있다. 따라서 작성한 다이어그램을 바탕으로 학습한 내용을 정리 하려고 한다.  

<br>

## SecurityFilterChain이 생성되는 과정
<img src="https://github.com/user-attachments/assets/50e5f4a9-3491-42cd-9371-f158dc839aed" alt="empty"
                    style="width: 100%; height: auto;">  

Spring Security는 서블릿 필터를 기반으로 생성된 SecurityFilterChain을 관리하고 있다. 이 SecurityFilterChain이 생성되는 과정은 Servlet 개념에 대해서 먼저 알고 있어야 한다.  

### Servlet과 Servlet Container
서블릿은 자바 기반의 웹 애플리케이션을 개발을 위한 서버 측 프로그래밍 기술이다. 서블릿은 클라이언트의 요청을 처리하고 적절한 응답을 생성하는 역할을 수행하는데, 이때 요청부터 응답까지의 비즈니스 로직을 제외한 모든 일련의 과정은 공용적으로 사용하기 때문에 문자열 구조를 모두 파싱하여 HttpServlet Request, Response 객체를 통해 사용할 수 있도록 처리한다.    

서블릿 컨테이너는 서블릿을 실행하고 관리하는 서버다. 클라이언트의 HTTP 요청을 받아 서블릿을 호출하고, 서블릿이 처리한 결과를 다시 클라이언트에게 응답으로 전달한다. 서블릿 컨테이너는 원활한 통신을 위한 소켓 통신 지원, 서블릿의 생명 주기 관리, 멀티쓰레드 지원 및 관리 등 여러 기능들을 제공한다.  

### FilterChain
![Image](https://github.com/user-attachments/assets/013aac35-acfb-4ead-8458-0d2b797f29aa)  

Spring Security는 웹 애플리케이션의 보안 필터 체인을 구성하여 인증, 인가, 요청의 검증 등을 처리한다. Spring Security는 단일 HTTP 요청을 처리하기 위해 계층적으로 구성된 FilterChain을 생성한다.  

필터 체인은 여러 개의 서블릿 필터와 하나의 서블릿으로 구성되어 있다. 필터 체인은 컨트롤러 단에서 처리하기 이전에 가로채서 먼저 수행하게 된다.  

구성된 하나의 서블릿 필터는 하위 필터 인스턴스나 서블릿이 호출되지 않도록 방지한다. 그리고 하위 필터 인스턴스와 서블릿이 사용하는 HttpServletRequest 또는 HttpServletResponse를 수정한다. 이를 통해 요청이나 응답에 대한 처리를 유연하게 변경할 수 있다.  

### DelegatingFilterProxy와 FilterChainProxy    
![Image](https://github.com/user-attachments/assets/1e16d4e4-0263-4430-a26e-c09812af1f5c)  

DelegatingFilterProxy는 ServletFilter의 구현체이다. 서블릿 컨테이너는 서블릿 필터를 관리하지만 스프링의 빈(Bean)은 인식하지 못하기 때문에 서블릿 필터 인스턴스를 실행할 수 없다. 이를 해결하기 위해 DelegatingFilterProxy가 Spring의 ApplicationContext에서 필터 역할을 하는 빈을 찾아서 필터 작업을 위임한다.  

FilterChainProxy는 여러 Security 필터들을 하나의 체인으로 묶어, HTTP 요청이 들어올 때 각 필터가 순차적으로 실행되도록 한다.  

### SecurityFilterChain과 Security Filters
SecurityFilterChain은 Spring Security에서 HTTP 요청에 대한 보안 처리를 담당하는 필터 체인을 정의하는 인터페이스이다. 이 체인은 여러 개의 보안 필터를 순차적으로 실행하게 관리한다.  

Security Filters는 요청 처리 중에 각 보안 작업(인증, 권한 부여, 세션 관리 등)을 수행하는 개별 필터들이다.  

<br>

## Spring Security 구조와 동작 원리
<img src="https://github.com/user-attachments/assets/76e0ba4e-f06f-4a02-9974-a683eb984ad3" alt="empty" style="width: 800px; height: auto;">   

사용자의 로그인 요청이 들어오면, 인증 과정과 처리 절차가 어떻게 이루어지는지 정리하려고 한다.  

### SecurityContextHolder
SecurityContextHolder는 SecurityContext 객체를 통해 사용자의 인증 정보(Authentication 객체)를 보관하며, 기본적으로 스레드 로컬(ThreadLocal)을 사용하여 각 스레드가 독립적으로 보안 정보를 관리할 수 있도록 한다.  

SecurityContext는 Authentication 객체를 포함하고 있으며, 이 객체는 사용자의 인증 상태와 관련된 정보를 제공한다.

### Authentication Filter
AuthenticationFilter는 HTTP 인증 요청을 가로채서, 이 인증이 필요한 요청인지 확인하는 것이다.  

이 필터는 AbstractAuthenticationProcessingFilter 클래스를 확장하여 사용된다. 여러 서브클래스가 AuthenticationFilter를 확장하여 각기 다른 인증 방식을 처리한다. 예를 들어, UsernamePasswordAuthenticationFilter, OAuth2LoginAuthenticationFilter 등이 있다.

### UsernamePasswordAuthenticationFilter
UsernamePasswordAuthenticationFilter는 AuthenticationFilter의 구체적인 서브클래스로, 아이디와 비밀번호를 사용한 기본적인 인증 방식을 처리한다.

1. 사용자가 사용자 이름과 비밀번호를 입력하면, UsernamePasswordAuthenticationFilter는 이를 추출하여 UsernamePasswordAuthenticationToken을 생성한다. 이 토큰은 Authentication 객체로, HTTP 요청에서 사용자 이름과 비밀번호를 추출하여 생성된다.
2. 생성된 UsernamePasswordAuthenticationToken은 인증을 위해 AuthenticationManager로 전달된다. AuthenticationManager는 설정된 사용자 정보 저장 방식에 따라 인증을 처리한다.
3. 인증에 실패하면, SecurityContextHolder는 비워지고, AuthenticationFailureHandler가 호출된다.
4. 인증에 성공하면, 인증된 사용자 정보는 SecurityContextHolder에 설정되며, 이후 애플리케이션에서 인증 정보를 사용할 수 있다.

### AuthenticationManager
사용자 인증 정보인 Authentication Filter를 확인하고 인증이 성공적인지 아닌지를 판별하는 인터페이스이다. 일반적인 구현체인 ProviderManager와 여러 개의 AuthenticationProvider를 결합하여 인증을 처리한다. 인증이 완료되면 Authentication 객체를 반환한다.  

1. 인증을 시도하는 사용자가 인증 정보를 제출한다. 
2. AuthenticationManager는 여러 AuthenticationProvider를 사용하여 인증을 시도한다. 
3. 각 AuthenticationProvider는 Authentication 객체를 받으며, 인증이 성공하면 인증된 사용자 정보를 담은 Authentication 객체를 반환한다. Authentication 객체는 SecurityContextHolder에 설정되어 현재 사용자의 인증 정보를 저장한다.  

### ProviderManager
AuthenticationManager의 일반적인 구현체이다. ProviderManager는 AuthenticationProvider 인스턴스들을 주입해서 관리하는데, 각 AuthenticationProvider가 인증이 성공했음을 알리거나 실패할 경우 AuthenticationProvider가 결정을 내리도록 위임한다.  

ProviderManager는 성공적인 인증 요청 후 반환된 Authentication 객체에서 민감한 자격 증명 정보를 지운다. 이렇게 함으로써 비밀번호와 같은 정보가 HttpSession에 불필요하게 오래 남지 않도록 방지한다.  

### AuthenticationProvider
AuthenticationProvider는 특정 유형의 인증을 수행하는 방법을 알고 있다. 예를 들어, DaoAuthenticationProvider는 데이터베이스에서 사용자 정보를 조회하여 인증을 처리하고, JwtAuthenticationProvider는 JWT 토큰을 이용한 인증을 처리한다.  

### DaoAuthenticationProvider
DaoAuthenticationProvider는 AuthenticationProvider 인터페이스를 구현한 클래스로, 데이터베이스 또는 다른 영속성 저장소에서 사용자 정보를 가져와 사용자 이름과 비밀번호를 인증하기 위해 UserDetailsService와 PasswordEncoder를 사용하는 인증 처리 구현체이다.

### UserDetailsService
UserDetailsService는 사용자가 로그인할 때 특정 사용자에 대한 세부 정보를 조회하고 반환하는 책임을 가진다. 주요 메서드인 loadUserByUsername(String username)는 특정 사용자 이름에 대한 사용자 정보를 반환하며, 반환되는 객체는 UserDetails이다.  

사용자는 UserDetailsService를 커스터마이즈하여 사용자 정의 인증을 구현할 수 있다.

### UserDetails
UserDetails는 사용자의 비밀번호, 권한, 계정 상태 등 인증에 필요한 정보를 제공한다. 

### DelegatingPasswordEncoder
DelegatingPasswordEncoder는 여러 종류의 비밀번호 인코딩 방식을 지원하는 PasswordEncoder의 구현체이다. 이 클래스는 다양한 인코딩 방식에 맞는 처리를 위임(delegate)하는 방식으로 작동한다.  

DelegatingPasswordEncoder는 여러 암호화 방식이 혼합되어 사용될 때 유용하며, 주로 BCrypt, Argon2, Pbkdf2, SCrypt 등을 지원한다.  

비밀번호 인코딩 방식은 적응형 단방향 해시 함수를 사용하여 CPU와 RAM 자원을 많이 소모하도록 설계되어 있어 보안성이 높다. 그중에서도 BCrypt는 Spring Security에서 가장 일반적으로 권장되는 비밀번호 해싱 방식이다. BCrypt는 패스워드 크래킹에 강력하게 저항하도록 설계되어 있어 선택하게 되었다.

### 전체적인 구조 흐름 정리
1. 인증 필터는 UsernamePasswordAuthenticationToken을 생성하여 AuthenticationManager로 전달한다.
2. AuthenticationManager는 ProviderManager로 구현되며, ProviderManager는 DaoAuthenticationProvider 유형의 AuthenticationProvider를 사용하도록 구성된다.
3. DaoAuthenticationProvider는 UserDetailsService를 통해 사용자의 정보를 조회한다.
4. DaoAuthenticationProvider는 PasswordEncoder를 사용하여 조회된 UserDetails의 비밀번호를 검증한다.
5. 인증이 성공하면, 반환된 Authentication 객체는 UsernamePasswordAuthenticationToken 유형이며, principal은 UserDetails이다.
6. 최종적으로 반환된 UsernamePasswordAuthenticationToken은 인증 필터에 의해 SecurityContextHolder에 설정된다.

<br>

# 참고 자료
- [Servlet Applications/Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Servlet Authentication/Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)