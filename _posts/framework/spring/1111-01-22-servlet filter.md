---
title: Servlet filter
categories: temp
---

# Servlet filter
![Image](https://github.com/user-attachments/assets/b1415e56-bbee-417b-a1d7-5c48567f9d11)  

servlet filter는 spring security 기반의 filter다. 클라이언트가 애플리케이션에 요청을 보내면, container는 요청 경로에 맞는 filter와 servlet을 포함하는 filterChain을 생성한다.  

servlet filter는 "web.xml"에서 구성하거나, spring Boot를 사용하는 경우 이를 bean "@Webfilter" 사용하여 일부로 구성할 수 있다.

**servlet**  
하나의 servlet은 httpServletRequest, httpServletResponse를 처리할 수 있고, 두 개 이상의 filter를 사용할 수 있다.  

## filterChain
```Java
// filterChain Usage Example
public void dofilter(ServletRequest request, ServletResponse response, filterChain chain) {
	// do something before the rest of the application
    chain.dofilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

filterChain은 filter가 호출되는 즉, 실행되는 순서에 따라 결과가 달라진다.  

### filterChain을 사용하는 목적
* HttpServletResponse 사용해서 후속 filter나 servlet이 호출되지 않도록 막을 수 있다.  
* 후속 filter나 servlet에서 HttpServletRequest이나 HttpServletResponse을 변경할 수 있다.

## DelegatingfilterProxy 
spring은 "DelegatingFilterProxy"를 제공한다. 이 proxy는 servletContainer의 lifecycle와 ApplicationContext를 연결하는 역할을 수행한다. servletContainer는 자체 표준을 사용해 filter 인스턴스를 등록할 수 있지만, spring에서 정의한 빈은 인식하지 못한다. DelegatingFilterProxy는 servletContainer의 표준 방식으로 등록할 수 있게 해주며, 실제 filter 작업을 spring bean으로 위임하여 처리한다.

**applicationContext ?**  
applicationContext는 설정 클래스로부터 bean 생성을 요청하고, 생성된 bean을 반환한다.

### DelegatingfilterProxy?
DelegatingfilterProxy는 servletFilterProxy이다. 실제 filter 작업을 spring 관리 bean에게 위임한다.
web.xml에서는 "targetBeanName" 필터 초기화 파라미터를 지원하며, 이를 사용하여 ApplicationContext에서 대상 빈의 이름을 지정할 수 있다. 단, 지정된 필터 이름은 spring의 root ApplicationContaxt에서 bean 이름과 일치해야 한다. 이후 DelegatingfilterProxy의 모든 호출은 springContaxt의 해당 bean으로 위임된다. 단, bean은 표준 servletFilter 인터페이스를 구현해야 한다.

**장점**
```
* 복잡한 설정이 필요한 필터 구현에 유용하다.
* filter 인스턴스에게 bean 정의 방식을 적용할 수 있게 한다.
* spring root applicationContext에서 서비스 bean을 조회하는 방법과 결합하여 표준 filter 설정을 고려할 수 있다.
```

**targetFilterLifecycle 관리** 
servletFilter의 lifeCycle 메서드는 기본적으로 대상 빈으로 위임되지 않으며, ApplicationContext가 관리한다.  
servletContainer가 lifeCycle을 관리하도록 하려면 "targetFilterLifecycle" 필터 초기화 파라미터를 "true"로 설정하고, 
init(jakarta.servlet.FilterConfig)와 destroy() 메서드가 대상 bean에서 호출되게 만들어야 한다.  

DelegatingfilterProxy는 servletContainer에서 인스턴스 기반으로 filter를 등록할 때 생성자 매개변수를 받을 수 있다. 
보통 WebApplicationInitializer SPI와 함께 사용된다.  제공된 생성자는 위임할 filterBean을 제공하거나, ApplicationContext와 bean 이름을 전달 받아, 
servletContext에서 ApplicationContext를 따로 조회할 필요 없이 요구된 filter를 가져올 수 있다.  

**WebApplicationInitializer ?** 
servlet 환경에서 servletContaxt를 프로그래밍 방식으로 설정할 수 있게 해 준다.    
이 인터페이스를 구현하면 "SpringServletContainerInitializer"가 자동으로 이를 감지하여 실행되며, servletContainer에 의해 자동으로 초기화된다.  

## DelegatingfilterProxy와 FilterChain의 상호작용
![Image](https://github.com/user-attachments/assets/0beb9fb1-83d7-4781-93f8-e9e2a89349a5)  

### 동작 과정
1. http 요청 시, 요청에 대한 filterChain을 생성한다.
2. 생성된 filterChain은 포함된 filter들을 순차적으로 호출한다. 이때, DelegatingFilterProxy는 ApplicationContext에서 지정된 filterBean을 찾아 호출하고, 이를 통해 요청을 위임하여 filter 로직을 실행한다.
3. filterChain 처리가 완료되면, 요청은 DispatcherServlet으로 전달되어 최종적으로 처리된다.  

```Java
// DelegatingFilterProxy Pseudo Code
public void dofilter(ServletRequest request, ServletResponse response, filterChain chain) {
	filter delegate = getfilterBean(someBeanName);
	delegate.dofilter(request, response);
}
```

### 의사 코드 동작 과정
1. **filter 지연 로딩**  
filter가 실제로 필요할 때 springContaxt에서 filterBean을 가져온다.  
2. **작업 위임**  
Bean에게 작업을 위임한다.  

### DelegatingfilterProxy의 지연로딩
DelegatingFilterProxy는 필터 빈 인스턴스를 나중에 조회할 수 있도록 해준다. 서블릿 컨테이너는 애플리케이션을 시작하기 전에 필터 인스턴스를 등록해야 하지만, Spring은 보통 ContextLoaderListener를 사용해 Spring 빈을 로드한다. 그러나 ContextLoaderListener는 필터를 등록해야 하는 시점 이후에 실행된다. 이를 해결하기 위해 DelegatingFilterProxy는 필터 빈을 지연 로딩하여 나중에 조회하고 사용하도록 한다.

## filterChainProxy
![Image](https://github.com/user-attachments/assets/fe660afd-a36c-4792-9629-7e52d983f6ee)
spring Security에서 servlet을 지원하는 기능은 filterChainProxy라는 객체에 포함되어 있다. filterChainProxy는 spring Security에서 제공하는 특별한 filter인데, 여러 개의 filter를 SecurityfilterChain을 통해 순차적으로 처리할 수 있게 해줍니다. filterChainProxy는 spring에서 Bean으로 관리되므로, 보통 DelegatingfilterProxy라는 또 다른 filter로 감싸서 사용합니다.

# SecurityfilterChain
![Image](https://github.com/user-attachments/assets/a1fbdad6-1aae-4b9e-81a6-c9265b508f3a)  
filterChainProxy를 구성하는 데 사용되는데, HttpServletRequest(즉, 웹 요청)를 검사하여, 해당 요청에 대해 이 체인이 적용될지 결정한다.
SecurityfilterChain은 filterChainProxy가 요청을 처리할 때, 어떤 보안 filter가 실행될지를 결정하는 역할을 합니다. 즉, 웹 요청이 들어오면, SecurityfilterChain이 어떤 filter가 적용될지 선택하게 되는

**filterChainProxy에**
spring Security에서 사용되는 보안 filter들은 보통 spring Bean으로 관리되지만, 이 filter들은 DelegatingfilterProxy 대신 filterChainProxy에 등록됩니다. filterChainProxy를 사용하는 이유는 여러 가지가 있는데, 그 중 하나는 spring Security의 servlet 기능을 시작하는 기본 지점이 되기 때문입니다. 그래서 spring Security의 servlet 기능에서 문제가 발생했을 때, filterChainProxy에 디버깅 포인트를 추가하는 것이 문제를 해결하는 좋은 방법이 됩니다.

filterChainProxy는 spring Security에서 중요한 역할을 하기 때문에 단순히 선택적인 작업을 넘어 필수적인 기능을 수행합니다. 예를 들어, 메모리 누수가 발생하지 않도록 SecurityContext를 정리하거나, 애플리케이션을 특정 공격으로부터 보호하기 위해 HttpFirewall을 적용하는 등의 일을 처리합니다.

servlet container에서는 filter가 URL을 기준으로 호출되지만, filterChainProxy는 더 유연하게 요청의 다른 요소들(예: 요청 헤더, 파라미터 등)을 기준으로 언제 filter를 호출할지 결정할 수 있다. 이를 가능하게 하는 것이 바로 RequestMatcher 인터페이스입니다.

![Image](https://github.com/user-attachments/assets/36b2f9d3-bd8d-440a-af0e-de0f14de7d7d)  

여러 개의 SecurityfilterChain이 있을 때, filterChainProxy는 요청된 URL에 대해 가장 먼저 일치하는 filter만 호출합니다. 예를 들어, /api/messages/라는 URL이 요청되면, /api/** 패턴에 맞는 첫 번째 filter(SecurityfilterChain0)만 실행됩니다. 만약 /messages/라는 URL이 요청되면, /api/**와 일치하지 않으므로 다음 filter들을 차례로 확인한 뒤, 마지막에 일치하는 filter(SecurityfilterChainn)가 실행됩니다.

SecurityfilterChain은 여러 개의 보안 filter를 결합할 수 있다.  SecurityfilterChain은 독립적으로 구성될 수 있으며, SecurityfilterChain은 보안 filter를 하나도 설정하지 않는다면, spring Security가 특정 요청을 아예 처리하지 않는다.

# Security filters
spring Security에서는 여러 보안 filter들이 SecurityfilterChain을 통해 filterChainProxy에 등록되어 순차적으로 실행됩니다. 이 filter들은 주로 공격 방지, 인증, 권한 부여 등 다양한 보안 작업을 수행합니다. 중요한 점은 이 filter들이 실행되는 순서가 매우 중요하다는 것입니다. 예를 들어, 인증을 먼저 처리하고, 그 후에 권한 부여를 처리해야 하는 경우처럼 말이죠. 보통 filter의 순서를 신경 쓸 필요는 없지만, 필요한 경우 filterOrderRegistration 코드에서 순서를 확인할 수 있다.

또한, 이 보안 filter들은 HttpSecurity를 사용하여 설정하는 경우가 많습니다. 이 부분을 더 자세히 살펴보기 위해, 예시로 보안 구성을 확인해보겠습니다.
```Java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityfilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults())
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            );

        return http.build();
    }

}
```
보안 filter들은 실행되는 순서에 따라 먼저 CSRF 공격을 막고, 그다음 인증을 처리한 후, 마지막으로 권한을 확인하는 방식으로 동작합니다.

# 보안 filter 출력 (Printing the Security filters)
```Java
2023-06-14T08:55:22.321-03:00  DEBUG 76975 --- [           main] o.s.s.web.DefaultSecurityfilterChain     : Will secure any request with [ DisableEncodeUrlfilter, WebAsyncManagerIntegrationfilter, SecurityContextHolderfilter, HeaderWriterfilter, Csrffilter, Logoutfilter, UsernamePasswordAuthenticationfilter, DefaultLoginPageGeneratingfilter, DefaultLogoutPageGeneratingfilter, BasicAuthenticationfilter, RequestCacheAwarefilter, SecurityContextHolderAwareRequestfilter, AnonymousAuthenticationfilter, ExceptionTranslationfilter, Authorizationfilter]
```
애플리케이션이 시작될 때, 보안 filter들이 호출된 순서대로 목록이 DEBUG 레벨로 로그에 출력됩니다. 이 로그를 통해, 자신이 추가한 보안 filter가 목록에 제대로 포함되었는지 확인할 수 있다.
애플리케이션에서 보안 filter들이 어떤 순서로 실행되는지 대략적으로 확인할 수 있지만, 더 나아가 각 요청마다 개별 filter가 실행되는지 확인할 수 있도록 로깅을 설정할 수도 있다. 이렇게 하면, 자신이 추가한 filter가 실제로 호출되었는지 확인하거나, 예외가 발생한 정확한 위치를 추적하는 데 도움이 됩니다.

# filterChain에 filter 추가하기 (Adding filters to the filter Chain)
기본적으로 제공되는 보안 filter들이 대부분의 상황에서 충분하지만, 특정 요구에 맞게 사용자 정의 filter를 추가하고 싶을 때가 있다. 이때 HttpSecurity를 사용하여 filter를 추가할 수 있는 세 가지 방법이 있다:
```
addfilterBefore: 특정 filter 앞에 추가.
addfilterAfter: 특정 filter 뒤에 추가.
addfilterAt: 특정 filter를 자신의 filter로 교체
```

# 사용자 정의 filter 추가하기 (Adding a Custom filter)
자신만의 filter를 만들 때, 그 filter가 filterChain에서 어느 위치에 있어야 할지 고려해야 합니다.
filterChain에서 발생하는 주요 이벤트들은 다음과 같습니다:

1. SecurityContext 로드: 사용자의 보안 정보가 세션에서 불러와집니다.
2. 보호 작업: 요청이 CSRF, CORS, 보안 헤더 등의 공격으로부터 보호됩니다.
3. 인증: 요청이 사용자인지 확인하는 인증 과정이 진행됩니다.
4. 권한 부여: 사용자가 요청을 수행할 권한이 있는지 확인하는 권한 부여가 이루어집니다.

![Image](https://github.com/user-attachments/assets/80528790-f364-40a1-b07a-15e5cd4a822d)  
자신의 filter가 filterChain에서 적절하게 실행되도록 하려면, filter가 실행될 때 어떤 작업들이 먼저 이루어져야 하는지를 고려해야 합니다. 예를 들어, 인증이 먼저 되어야 filter가 실행될 수 있다면, filter의 위치를 인증 후에 배치해야 합니다.

애플리케이션에서 보통 사용자 정의 인증을 추가하는 경우, 해당 인증 filter는 Logoutfilter 이후에 배치되어야 합니다. 이는 인증이 로그아웃 처리 이후에 이루어져야 하기 때문입니다.

```Java
import java.io.IOException;

import jakarta.servlet.filter;
import jakarta.servlet.filterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.ServletRequest;
import jakarta.servlet.ServletResponse;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import org.springframework.security.access.AccessDeniedException;

public class Tenantfilter implements filter {
    @Override
    public void dofilter(ServletRequest servletRequest, ServletResponse servletResponse, filterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        String tenantId = request.getHeader("X-Tenant-Id"); (1)
        boolean hasAccess = isUserAllowed(tenantId); (2)
        if (hasAccess) {
            filterChain.dofilter(request, response); (3)
            return;
        }
        throw new AccessDeniedException("Access denied"); (4)
    }
}
```
이 경우, 요청 헤더에 포함된 테넌트 ID를 확인하고, 그 테넌트에 대한 접근 권한을 가진 사용자인지 검사하는 filter를 추가하려는 상황입니다. 이 filter는 테넌트 기반의 접근 제어를 구현할 때 유용할 수 있다.
```
테넌트 ID 가져오기: 요청 헤더에서 테넌트 ID를 추출합니다.
접근 권한 확인: 현재 사용자가 이 테넌트에 접근할 권한이 있는지 확인합니다.
filter 계속 실행: 권한이 있으면 요청을 계속 처리하며, 나머지 보안 filter들을 실행합니다.
접근 거부: 권한이 없다면 AccessDeniedException을 던져 요청을 차단합니다.
```

**OncePerRequestfilter**
OncePerRequestfilter는 한 요청에 대해 단 한 번만 실행되는 filter를 만들 때 사용하는 클래스입니다. 이 클래스를 확장하면, filter를 직접 구현할 필요 없이 dofilterInternal 메서드를 오버라이드하여 요청과 응답을 처리할 수 있다. 이 방법은 주로 요청당 한 번만 실행되는 filter가 필요할 때 사용됩니다.

```Java
@Bean
SecurityfilterChain filterChain(HttpSecurity http) throws Exception {
    http
        // ...
        .addfilterAfter(new Tenantfilter(), AnonymousAuthenticationfilter.class);
    return http.build();
}
```
filter를 SecurityfilterChain에 추가하려면, 해당 filter가 인증 filter들 뒤에 위치해야 합니다. 이는 filter가 현재 사용자에 대한 정보를 확인해야 하기 때문에, 사용자가 인증된 후에 실행되어야 한다는 의미입니다. 따라서, 인증 filter들이 완료된 후에 이 filter가 실행될 수 있도록 filterChain에 배치해야 합니다.

AnonymousAuthenticationfilter는 인증 filterChain에서 마지막으로 실행되는 filter이므로, 이 filter 뒤에 새로 만든 filter를 추가해야 합니다. 이렇게 하면 사용자가 인증된 후에 추가적인 작업을 진행할 수 있다.

HttpSecurity 객체의 addfilterAfter 메서드를 사용하여 Tenantfilter를 AnonymousAuthenticationfilter 뒤에 추가하는 방법을 의미합니다. 이렇게 하면 Tenantfilter가 AnonymousAuthenticationfilter 이후에 실행되도록 보장할 수 있다.

AnonymousAuthenticationfilter는 인증되지 않은 사용자를 위한 기본 인증 처리를 담당합니다.
Tenantfilter는 인증이 끝난 후 사용자가 특정 테넌트에 접근할 수 있는지 확인하는 역할을 합니다. 따라서 인증이 완료된 후에 실행해야 하므로 addfilterAfter로 AnonymousAuthenticationfilter 뒤에 위치시킵니다.

# filter를 Bean으로 선언하기 (Declaring Your filter as a Bean)
```Java
@Bean
public filterRegistrationBean<Tenantfilter> tenantfilterRegistration(Tenantfilter filter) {
    filterRegistrationBean<Tenantfilter> registration = new filterRegistrationBean<>(filter);
    registration.setEnabled(false);
    return registration;
}
```
이렇게 하면 HttpSecurity만 추가됩니다.

filter를 spring bean으로 선언하면 (@Component로 주석을 달거나 설정 파일에 bean으로 선언하는 방식), spring Boot는 이를 자동으로 내장 container에 등록합니다. 이렇게 되면 filter가 두 번 호출될 수 있다. 한 번은 container에 의해서, 또 한 번은 spring Security에 의해서 호출되며, 호출 순서도 다를 수 있다.
이 때문에, 많은 경우 filter는 spring bean으로 선언되지 않습니다.
그러나 filter가 spring bean으로 선언되어야 하는 경우 (예: 의존성 주입을 사용해야 하는 경우), spring Boot에게 container에 filter를 등록하지 않도록 하려면 filterRegistrationBean bean을 선언하고, enabled 속성을 false로 설정하면 됩니다.

# spring Security filter 사용자 정의
```Java
@Bean
SecurityfilterChain filterChain(HttpSecurity http) throws Exception {
	http
		.httpBasic(Customizer.withDefaults())
        // ...

	return http.build();
}
```
```
http.addfilter() 메서드는 filter를 추가하는 데 사용됩니다.
BasicAuthenticationfilter는 기본적인 HTTP 인증을 처리하는 filter로, 이와 같은 방식으로 filter를 구성할 수 있다.
```
일반적으로, filter의 DSL 메서드를 사용하여 spring Security의 filter를 구성할 수 있다. 예를 들어, 가장 간단한 방법으로 BasicAuthenticationfilter를 추가하는 방법은 DSL을 통해 추가하는 것입니다

```Java
@Bean
SecurityfilterChain filterChain(HttpSecurity http) throws Exception {
	BasicAuthenticationfilter basic = new BasicAuthenticationfilter();
	// ... configure

	http
		// ...
		.addfilterAt(basic, BasicAuthenticationfilter.class);

	return http.build();
}
```
spring Security filter를 직접 구성하려는 경우 다음과 같이 addfilterAt를 사용하여 DSL에 지정하면 됩니다.

```Java
@Bean
SecurityfilterChain filterChain(HttpSecurity http) throws Exception {
	BasicAuthenticationfilter basic = new BasicAuthenticationfilter();
	// ... configure

	http
		.httpBasic(Customizer.withDefaults())
		// ... on no! BasicAuthenticationfilter is added twice!
		.addfilterAt(basic, BasicAuthenticationfilter.class);

	return http.build();
}
```
spring Security에서는 filter가 이미 추가된 경우, 동일한 filter를 두 번 추가하려고 하면 예외가 발생합니다. 예를 들어, httpBasic() 메서드를 사용하면 BasicAuthenticationfilter가 자동으로 추가되므로, 동일한 filter를 다시 추가하려고 할 경우 충돌이 발생하고 예외가 던져집니다.

이 경우 BasicAuthenticationfilter를 직접 구성하고 있으므로 httpBasic에 대한 호출을 제거하세요.

**특정 filter가 추가되지 않게 하는 함수형 방법**
```Java
.httpBasic((basic) -> basic.disable())
```
HttpSecurity에서 특정 filter가 추가되지 않도록 재구성할 수 없는 경우, 보통 해당 spring Security filter를 비활성화하는 방법은 해당 DSL의 disable 메서드를 호출하는 것입니다. 

# Handling Security Exceptions
![Image](https://github.com/user-attachments/assets/22f5ab2b-a6da-4fb8-b11c-f1fe9ab9d266)

**ExceptionTranslationfilter**
ExceptionTranslationfilter는 spring Security에서 인증 및 권한 관련 예외를 처리하는 filter입니다. 이 filter는 두 가지 주요 예외인 AccessDeniedException (권한 거부 예외)과 AuthenticationException (인증 예외)을 잡아, 이를 HTTP 응답으로 변환합니다. 예를 들어, 사용자가 인증되지 않았거나 권한이 없을 때 적절한 HTTP 상태 코드(예: 401 Unauthorized, 403 Forbidden)를 반환하도록 도와줍니다.

이 filter는 filterChainProxy에 삽입되어, spring Security의 다른 보안 filter와 함께 동작합니다.

1. 먼저, ExceptionTranslationfilter는 filterChain.dofilter(request, response)를 호출하여 애플리케이션의 나머지 부분을 호출합니다.

2. 사용자가 인증되지 않았거나 AuthenticationException이 발생하는 경우 인증을 시작합니다.
SecurityContextHolder 비우기:
인증과 관련된 정보가 SecurityContextHolder에 저장되는데, 예외가 발생하거나 인증이 필요한 상황에서는 이 정보를 비웁니다. 이는 보안 Contaxt를 초기화하거나 재설정하기 위한 과정입니다.

HttpServletRequest 저장:
HttpServletRequest는 인증이 성공하면 원래의 요청을 재시도할 수 있도록 저장됩니다. 예를 들어, 인증되지 않은 사용자가 보호된 페이지에 접근하려 할 때, 인증이 완료되면 원래의 요청을 다시 수행하도록 처리할 수 있다.

AuthenticationEntryPoint 사용:
AuthenticationEntryPoint는 인증이 필요한 요청에 대해 클라이언트에게 인증 정보를 요구합니다. 예를 들어, 로그인 페이지로 리디렉션하거나, HTTP 응답에 WWW-Authenticate 헤더를 추가해 클라이언트에게 인증을 요청할 수 있다.
3. AccessDeniedException 처리:
AccessDeniedHandler는 AccessDeniedException을 처리하고, 일반적으로 403 상태 코드를 반환하거나, 사용자에게 권한이 부족하다는 메시지를 전달하는 역할

ExceptionTranslationfilter는 기본적으로 **AccessDeniedException**이나 AuthenticationException 예외가 발생할 때만 동작합니다. 만약 애플리케이션에서 이러한 예외가 발생하지 않으면, 이 filter는 아무런 동작을 하지 않고 요청을 계속해서 filterChain에 전달합니다.

```Java
/// ExceptionTranslationfilter pseudocode
try {
	filterChain.dofilter(request, response);
} catch (AccessDeniedException | AuthenticationException ex) {
	if (!authenticated || ex instanceof AuthenticationException) {
		startAuthentication();
	} else {
		accessDenied();
	}
}
```
1. filterChain.dofilter가 호출되어 filterChain과 애플리케이션의 나머지 부분이 실행됩니다.
2. 인증되지 않았거나 인증 오류가 발생한 경우, 인증 프로세스를 시작합니다.
3. 인증된 사용자라도 권한이 없으면 "Access Denied" 메시지를 반환합니다.

1. filterChain에서 예외 처리
filterChain.dofilter(request, response)를 호출하는 것은 filterChain 내에서 나머지 애플리케이션을 실행하는 것과 동일합니다.
따라서 애플리케이션의 다른 부분, 예를 들어 filterSecurityInterceptor나 메서드 보안에서 AuthenticationException (인증 실패)이나 AccessDeniedException (권한 부족)이 발생하면, 해당 예외들이 **ExceptionTranslationfilter**에 의해 잡혀서 처리됩니다.
2. 사용자가 인증되지 않았거나 인증 오류가 발생하면 인증 시작
만약 사용자가 인증되지 않았거나, **AuthenticationException**이 발생한 경우, 이는 사용자가 인증되지 않았거나 잘못된 인증 정보를 제공한 경우입니다.
이때 인증 프로세스를 시작해야 합니다. 즉, 사용자가 인증을 요구하는 리소스를 요청했을 때, AuthenticationEntryPoint가 호출되어 로그인 페이지로 리디렉션하거나 인증을 요청합니다.
3. 그렇지 않으면 접근 거부 (Access Denied)
만약 사용자가 인증되었으나, 해당 리소스에 대한 권한이 부족한 경우, 즉 AccessDeniedException이 발생한 경우에는 **"Access Denied"**가 발생하여 리소스에 대한 접근이 거부됩니다.
이때 403 Forbidden 상태 코드가 반환되어, 사용자에게 접근 거부 메시지를 전달합니다.

# 인증 간 요청 저장

# RequestCache
```Java
@Bean
DefaultSecurityfilterChain springSecurity(HttpSecurity http) throws Exception {
	HttpSessionRequestCache requestCache = new HttpSessionRequestCache();
	requestCache.setMatchingRequestParameterName("continue");
	http
		// ...
		.requestCache((cache) -> cache
			.requestCache(requestCache)
		);
	return http.build();
}
```
이 문장은 spring Security에서 **RequestCache**를 어떻게 활용하여 인증되지 않은 사용자가 원래 요청한 리소스로 리디렉션되는지 설명합니다. RequestCache는 요청을 저장하고, 인증 후 그 요청을 복원하는 역할을 합니다.

인증되지 않은 요청:
사용자가 인증이 필요한 리소스를 요청하면, ExceptionTranslationfilter가 이를 감지하고 AuthenticationException을 발생시킵니다. 이때, ExceptionTranslationfilter는 원래 요청을 RequestCache에 저장합니다.
사용자 인증 후:
사용자가 성공적으로 인증하면, RequestCacheAwarefilter는 저장된 요청을 RequestCache에서 꺼내와서 사용자가 원래 요청한 리소스로 리디렉션합니다.

spring Security는 기본적으로 HttpSessionRequestCache를 사용하여 사용자가 인증되지 않은 상태에서 요청한 리소스를 저장하고, 인증 후 해당 요청을 복원합니다. 하지만 때때로 사용자 정의 RequestCache를 사용하고 싶을 수 있다. 이 예제에서는 continue라는 파라미터가 존재하는 경우에만 요청을 저장하는 방식으로 RequestCache를 사용자 정의하는 방법을 설명합니다.

사용자가 인증되지 않은 상태에서 보호된 리소스를 요청하면, 요청이 HttpSessionRequestCache에 저장됩니다.
사용자가 인증 후 원래 요청한 리소스로 리디렉션됩니다.

# 참고
- [Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html) 
- [Class DelegatingfilterProxy](https://docs.spring.io/spring-framework/docs/6.2.1/javadoc-api/org/springframework/web/filter/DelegatingfilterProxy.html)
- [Interface SecurityfilterChain](https://docs.spring.io/spring-security/reference/api/java/org/springframework/security/web/SecurityfilterChain.html)