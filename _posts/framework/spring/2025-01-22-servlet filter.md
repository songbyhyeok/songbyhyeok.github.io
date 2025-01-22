---
title: Servlet Filter
categories: spring
---
![Image](https://github.com/user-attachments/assets/b1415e56-bbee-417b-a1d7-5c48567f9d11)  

ApplicationContext는 설정 클래스로부터 빈 생성을 요청하고, 생성된 빈을 반환

servlet filter는 spring security 기반이다.
클라이언트가 애플리케이션에 요청을 보내면, 컨테이너는 요청 경로에 맞는 필터와 서블릿을 포함하는 FilterChain을 생성한다.

하나의 servlet은 HttpServletRequest, HttpServletResponse를 처리할 수 있고, 두 개 이상의 Filter를 사용할 수 있다.

필터 체인을 사용하는 목적
* HttpServletResponse 사용해서 후속 필터나 서블릿이 호출되지 않도록 막을 수 있다.  
* 필터는 후속 필터나 서블릿에서 사용할 요청(HttpServletRequest)이나 응답(HttpServletResponse)을 변경할 수 있다.

```Java
// FilterChain Usage Example
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	// do something before the rest of the application
    chain.doFilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```
필터는 다른 필터나 서블릿에만 영향을 주므로, 필터가 호출되는 순서가 매우 중요하다. 
실행되는 순서에 따라 결과가 달라진다. 

# DelegatingFilterProxy 
DelegatingFilterProxy는 실제 필터 작업을 Spring 관리 빈으로 위임하는 역할을 하는 필터이다.  
이 필터는 필터 체인(FilterChain) 내에서 다른 필터들과 함께 동작한다.

Spring에서 관리하는 빈으로 실제 필터 작업을 위임하는 서블릿 필터 프록시  
설정에서 "targetBeanName"을 설정하여, Spring 애플리케이션 컨텍스트 내 특정 빈으로 필터 작업을 위임할 수 있다.  

이 방법은 설정이 복잡한 필터를 사용할 때 특히 유용하다. Spring의 빈 정의 시스템을 필터 인스턴스에 적용하여 필터를 더 쉽게 관리하고 설정할 수 있게 해준다. 또 다른 방법은 Spring 루트 애플리케이션 컨텍스트에서 서비스 빈을 조회하면서 표준 필터 설정을 사용하는 것, 이렇게 하면 Spring 빈을 필터 설정과 결합할 수 있다.

targetFilterLifecycle true  
라이프사이클 메서드는 기본적으로 Spring 애플리케이션 컨텍스트에서 관리된다. 서블릿 컨테이너가 필터의 라이프사이클을 관리하도록 하려면(빈 등록)하려면 targetFilterLifecycle 초기화 파라미터를 "true"로 설정하고, Filter.init()과 Filter.destroy() 메서드가 대상 빈에서 호출되게 만들어야 한다.  

DelegatingFilterProxy는 서블릿 컨테이너에서 인스턴스 기반으로 필터를 등록할 때 생성자 매개변수를 받을 수 있습니다. 보통 WebApplicationInitializer와 함께 사용되며, 생성자에서 필터 빈을 직접 제공하거나 애플리케이션 컨텍스트와 빈 이름을 전달하여 필터를 가져올 수 있습니다. 이렇게 하면 서블릿 컨텍스트에서 애플리케이션 컨텍스트를 따로 조회할 필요가 없다.

DelegatingFilterProxy는 서블릿 컨테이너에서 인스턴스 기반으로 필터를 등록할 때 생성자 매개변수를 받을 수 있다. 보통 WebApplicationInitializer와 함께 사용되며, 생성자에서 필터 빈을 직접 제공하거나 애플리케이션 컨텍스트와 빈 이름을 전달하여 필터를 가져올 수 있다. 이렇게 하면 서블릿 컨텍스트에서 애플리케이션 컨텍스트를 따로 조회할 필요가 없다.

WebApplicationInitializer
서블릿 환경에서 서블릿 컨텍스트를 프로그래밍 방식으로 설정할 수 있게 해 준다.
이 인터페이스를 구현하면 SpringServletContainerInitializer가 자동으로 이를 감지하여 실행, 
SpringServletContainerInitializer는 서블릿 컨테이너에 의해 자동으로 초기화된다.

```Java
Example
The traditional, XML-based approach
Most Spring users building a web application will need to register Spring's DispatcherServlet. For reference, in WEB-INF/web.xml, this would typically be done as follows:
 <servlet>
   <servlet-name>dispatcher</servlet-name>
   <servlet-class>
     org.springframework.web.servlet.DispatcherServlet
   </servlet-class>
   <init-param>
     <param-name>contextConfigLocation</param-name>
     <param-value>/WEB-INF/spring/dispatcher-config.xml</param-value>
   </init-param>
   <load-on-startup>1</load-on-startup>
 </servlet>

 <servlet-mapping>
   <servlet-name>dispatcher</servlet-name>
   <url-pattern>/</url-pattern>
 </servlet-mapping>
The code-based approach with WebApplicationInitializer
Here is the equivalent DispatcherServlet registration logic, WebApplicationInitializer-style:
 public class MyWebAppInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
      XmlWebApplicationContext appContext = new XmlWebApplicationContext();
      appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

      ServletRegistration.Dynamic dispatcher =
        container.addServlet("dispatcher", new DispatcherServlet(appContext));
      dispatcher.setLoadOnStartup(1);
      dispatcher.addMapping("/");
    }

 }
```

# DelegatingFilterProxy와 FilterChain의 관계
![Image](https://github.com/user-attachments/assets/0beb9fb1-83d7-4781-93f8-e9e2a89349a5)  

전체 로직
클라이언트 http 요청 -> 요청에 대한 필터 체인 생성(여러 개 필터 포함) -> 필터 실행(
필터 체인에 있는 각 필터는 요청을 처리하고, DelegatingFilterProxy는 Spring 애플리케이션 컨텍스트에서 지정된 필터 빈을 찾아 이를 호출과 빈에게 요청을 위임하고 필터 로직을 실행) -> 
서블릿 처리: 필터 체인이 끝나면, 요청은 서블릿(예: DispatcherServlet)으로 전달되어 최종적으로 처리.

빈을 찾고 호출하기까지
elegatingFilterProxy는 먼저 Spring의 ApplicationContext에서 Filter0이라는 빈을 찾아낸 후, 그 빈을 호출하여 필터 작업을 수행한다.
```Java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
	Filter delegate = getFilterBean(someBeanName);
	delegate.doFilter(request, response);
}
```

1. 필터 지연 로딩
필터가 실제로 필요할 때 Spring 컨텍스트에서 필터 빈을 가져온다.
2. 작업 위임
Bean에 작업을 위임

장점
DelegatingFilterProxy의 또 다른 장점은 필터 빈 인스턴스를 지연 로딩할 수 있다는 것, 서블릿 컨테이너는 애플리케이션을 시작하기 전에 필터 인스턴스를 등록해야 하지만, Spring은 보통 ContextLoaderListener를 사용하여 Spring 빈을 로드한다. 이 로딩은 필터 인스턴스가 이미 등록된 후에 이루어지기 때문에, DelegatingFilterProxy는 필터 빈을 나중에 로딩할 수 있게 해준다. 이를 통해 Spring 빈이 로드된 후 필터 작업을 올바르게 처리할 수 있다.




# 참고
- [Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html) 
- [Class DelegatingFilterProxy](https://docs.spring.io/spring-framework/docs/6.2.1/javadoc-api/org/springframework/web/filter/DelegatingFilterProxy.html)