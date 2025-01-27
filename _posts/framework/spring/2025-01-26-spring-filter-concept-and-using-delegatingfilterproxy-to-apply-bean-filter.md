---
title: Spring Filter 개념과 그리고 DelegatingFilterProxy를 사용한 Bean 필터 적용 과정
categories: spring
---

# Servlet filter
<img src="https://github.com/user-attachments/assets/b1415e56-bbee-417b-a1d7-5c48567f9d11" style="width: 30%; height: auto;">  

클라이언트가 요청을 보내면, Spring은 이 요청을 처리할 여러 필터와 DispatcherServlet을 설정한다. 이들은 FilterChain이라고 불리는 연결된 체인 안에 포함되어, 요청을 순차적으로 처리하게 된다. 하나의 요청은 한 서블릿만 처리할 수 있지만, 그 요청을 처리하는 데 여러 개의 필터가 사용될 수 있다.

필터의 역할은 크게 두 가지이다.  
1. 요청 처리 과정 중 필터가 서블릿이나 다른 필터의 개입을 차단하고, 대신 필터가 직접 응답을 생성하여 반환한다.
2. 필터는 요청과 응답을 수정할 권한이 있으며, 이를 수정한 후 하위 필터나 서블릿에 전달할 수 있다.

**FilterChain**  
```Java
// filterChain Usage Example
public void dofilter(ServletRequest request, ServletResponse response, filterChain chain) {
	// do something before the rest of the application
    chain.dofilter(request, response); // invoke the rest of the application
    // do something after the rest of the application
}
```

체인필터는 하위 필터 인스턴스와 서블릿에만 영향을 미친다. 호출 순서가 요청 처리 흐름을 결정짓는 중요한 요소이므로, 이를 정확하게 설정하는 것이 중요하다.

## DelegatingfilterProxy
<img src="https://github.com/user-attachments/assets/0beb9fb1-83d7-4781-93f8-e9e2a89349a5" style="width: 30%; height: auto;">  

DelegatingfilterProxy는 servletFilterProxy이다. 이 필터는 ServletContainer와 Spring의 ApplicationContext를 연결한다.  

**ServletContainer**는 웹 서버와 통신하여 HTTP 요청을 받고, 서블릿을 사용하여 요청을 처리한 후 응답을 클라이언트에게 전송한다. 이 과정에서 서블릿 생명 주기 관리를 담당하는데, 이는 applicationContext와 연관되어 있다.  

**ApplicationContext**는 spring 컨테이너 IOC의 구현체이다. 클래스로부터 bean 생성을 요청하고, 생성된 bean을 반환하는 역할을 수행한다.  

### DelegatingFilterProxy는 필터 요청을 빈에 위임한다
ServletContainer는 자체 표준을 사용해 filter 인스턴스를 등록할 수 있지만, spring에서 정의한 빈은 인식하지 못하여 사용할 수 없다. 바로 이때, DelegatingFilterProxy가 중간에서 역할을 수행하여, Spring 빈을 표준 방식으로 등록하고 사용할 수 있게 해준다. 즉, 스프링에서 필터를 사용하려면 빈 객체를 사용해야 하는데, DelegatingFilterProxy가 ApplicationContext에서 빈을 찾아 요청 처리를 위임하는 방식으로 동작한다. 

```Java
// DelegatingFilterProxy Pseudo Code
public void dofilter(ServletRequest request, ServletResponse response, filterChain chain) {
	filter delegate = getfilterBean(someBeanName);
	delegate.dofilter(request, response);
}
```

위 의사 코드는 DelegatingFilterProxy가 **지연 로딩**을 통해 ApplicationContext에게 빈 필터를 요청 및 가져와서 빈에게 위임까지의 코드이다.  

### DelegatingfilterProxy의 지연로딩
DelegatingFilterProxy는 필터 등록과 Spring Bean 로딩 사이의 시점 차이를 해결한다. 필터는 서블릿 컨테이너에 먼저 등록되지만, 실제 필터 Bean은 Spring 컨텍스트가 ContextLoaderListener를 사용하여 준비된 후에 로딩하게 된다. 즉, 필터와 Spring Bean의 초기화 시점이 충돌하지 않도록 이를 조정한다.  

![Image](https://github.com/user-attachments/assets/c174acc7-9925-4f25-bb7e-65d632ae49d6)  
<br>

# 참고
- [Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html) 
- [스프링과 서블릿 컨테이너의 기본 개념](https://f-lab.kr/insight/understanding-spring-servlet-container-differences?gad_source=1&gclid=CjwKCAiAtNK8BhBBEiwA8wVt9zo9384rKu0V2YzuPhpzSeQsPhm4SFa14BICuSMXB8z4MGlyxxDMiBoCVCoQAvD_BwE)