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
<img src="https://github.com/user-attachments/assets/220d97b1-84df-4fdb-a032-21e850033339" alt="empty"
                    style="width: 100%; height: auto;">  

사용자의 로그인 요청이 들어오면, 인증 과정과 처리 절차가 어떻게 이루어지는지 정리하려고 한다.  

### SecurityContextHolder
SecurityContextHolder는 Spring Security에서 사용자의 보안 정보를 저장하고 제공하는 클래스이다. 주로 인증된 사용자의 세션 정보를 관리하며, 현재 실행 중인 스레드에서 보안 관련 정보를 가져오는 데 사용된다. SecurityContextHolder는 SecurityContext 객체를 통해 사용자의 인증 정보(Authentication 객체)를 보관하며, 기본적으로 스레드 로컬(ThreadLocal)을 사용하여 각 스레드가 독립적으로 보안 정보를 관리할 수 있도록 한다.  

### Authentication Filter

### AuthenticationManager

### ProviderManager

### AuthenticationProvider

### DaoAuthenticationProvider

### UserDetailsService

### DelegatingPasswordEncoder







<br>

# 참고 자료
- [Servlet Applications/Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Servlet Authentication/Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)