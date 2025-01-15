---
title: HandlerInterceptor 
categories: spring
---

# HandlerInterceptor
클라이언트의 요청을 처리할 핸들러가 실행되기 전에 **(Interceptor 하여)**, 특정 작업을 수행할 수 있도록 서버에 제공하는 인터페이스다. 

## Interceptor?
![Interceptor VSD_Example](https://github.com/user-attachments/assets/9e725c1d-c511-46ea-b940-f84cf7ae7cbe)  
출처: [위키피디아](https://en.wikipedia.org/wiki/Interceptor_pattern)  

사전 의미상 "가로채는" 라는 뜻을 가진 인터셉터는 처리 주기를 변경하거나 증강하는 방법을 제공하고자 할 때 사용되는 디자인패턴을 말하며, 
변경 사항이 투명하고 자동화 이점 때문에 프레임워크에서 일반적으로 사용한다.  

## 특징
핸들러 인터셉터는 구조 특성상 요청 핸들러 전후에 다양한 기능을 개입시킬 수 있다. 즉, 요청이 핸들러에 도달하기 전이나 응답이 클라이언트로 돌아가기 전, 후에 추가적인 작업을 할 수 있게 해 준다. 
이를 통해 미리 필요한 설정을 적용하거나 특정 로직을 추가 및 변경함으로써 핸들러의 실행 환경을 효과적으로 관리할 수 있다.  

핸들러 인터셉터 여러 핸들러에서 공통적으로 필요한 로직(예: 인증, 권한 체크, 로깅, 세션 처리 등)을 각각의 핸들러에서 직접 작성하는 대신, 인터셉터를 통해 일괄 처리할 수 있다. 이렇게 함으로써 
코드를 간결하게 유지하고 재사용성을 높이며, 유지보수가 용이해진다. 또한, 다양한 인터셉터를 조합하여 요청 처리 흐름을 세밀하게 제어할 수 있기 때문에 유연하다.

## 주의
Spring 5.3 이전까지는 HandlerInterceptorAdapter 클래스를 사용하여 간편하게 인터셉터를 생성할 수 있었지만, 
이후부터는 사용 중지(예정) 되었기 때문에, HandlerInterceptor 인터페이스를 직접 구현하는 방식을 권장하고 있다.  

## 정확히 언제 호출되는 걸까?
Spring 인터셉터가 어떻게 작동하는지 이해하기 위해서는 HandlerMapping, HandlerAdapter의 동작을 이해해야 한다.

### HandlerMapping과 HandlerAdapter
![다운로드](https://github.com/user-attachments/assets/f9b1e2b3-37f4-4e9d-af65-3c268a352839)  
출처: 김영한 MVC  

1. **HandlerMapping**는 handler method를 URL에 매핑한다. 
2. **DispatcherServlet**는 **HandlerAdapter**에게 매핑 핸들러 호출 명령을 내린다. 

1번과 2번 사이, 즉 요청 핸들러 호출 이전에 **HandlerInterceptor**는 요청을 가로채서 특정 작업을 먼저 수행한다. 이 작업은 **preHandle API**를 통해 이루어지며, 
반환 값이 true일 경우 요청 핸들러 처리 이후에, 인터셉터가 개입할 수 있는 제어권을 제공한다. 

## 구성과 등록
### 인터셉터 구성
```Java
@RestController
@RequestMapping("/api/v1")
public class InterceptorController implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, 
    Object handler) throws Exception {
        System.out.println("Before handling the request");
        return true;  // true를 반환하면 다음 인터셉터나 컨트롤러로 요청이 계속 전달된다.
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
    Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("After handling the request, but before rendering the view");
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
    Object handler, Exception ex) throws Exception {
        System.out.println("After the complete request and response");
    }

    @GetMapping("/hello")
    public String sayHello() {
        return "Hello, World!";
    }

    @PostMapping("/hello")
    public String postHello(@RequestBody String message) {
        return "Received message: " + message;
    }
}
```
인터셉터는 **HandlerInterceptor** 인터페이스를 구현하면, 세 가지 주요 메서드를 제공한다.
* **preHandle**:
요청이 컨트롤러에 전달되기 전에 호출된다..
여기에서 요청을 가로채고 처리할 수 있다. 예를 들어, 인증 체크를 할 수 있다.
return true를 반환하면 요청이 계속 처리되고, false를 반환하면 요청 처리가 중단된다.

* **postHandle**:
요청이 컨트롤러에서 처리된 후, 뷰가 렌더링되기 전에 호출된다.
예를 들어, 요청 처리 후 응답에 대해 추가적인 작업을 할 수 있다. (예: 추가 데이터 수정)

* **afterCompletion**:
요청 처리 후, 뷰 렌더링이 끝난 후 호출된다.
응답이 클라이언트로 전송된 후, 로그를 기록하거나 리소스를 해제하는 등의 작업을 할 수 있다.

### 인터셉터 등록 
```Java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new InterceptorController())
                .addPathPatterns("/**")  // 인터셉터를 적용할 URL 패턴 지정
                .excludePathPatterns("/login", "/register");  // 특정 URL은 제외
    }
}
```
인터셉터를 사용하려면 **WebMvcConfigurer** 인터페이스의 **addInterceptors** 메서드를 통해 인터셉터를 등록해야 한다.  

**API**: [Interface HandlerInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)

## 인터셉터 체인을 통한 핸들러 매핑 구성하기
```Java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public MyInterceptor1 myInterceptor1() {
        return new MyInterceptor1();
    }

    @Bean
    public MyInterceptor2 myInterceptor2() {
        return new MyInterceptor2();
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myInterceptor1()) // 첫 번째 인터셉터
                .addPathPatterns("/**"); // 특정 경로에만 인터셉터 적용
        registry.addInterceptor(myInterceptor2()) // 두 번째 인터셉터
                .addPathPatterns("/**");
    }
}
```
**--여러 핸들러에 동일한 인터셉터 체인을 적용한 설정--**  

설정은 xml이나 config 둘 중 하나를 선택하면 된다.  
인터셉터 체인은 HandlerMapping 빈에 대해 정의되며, 그 세부 수준을 공유하기 때문에 Config에서 제어가 가능하다. 위 코드와 같이 인터셉터들을 빈 객체로 등록시켜야 한다.

### 인터셉터 체인(interceptor chain)?
체인(Chain)은 여러 개의 개별적인 작업이나 요소들이 연결되어 하나의 흐름을 형성하는 구조를 말하는데, 
이를 여러 개의 인터셉터를 순차적으로 연결시켜, 요청 처리 전후에 실행되는 일련의 단계를 형성할 수 있다.
```
1. 첫 번째 인터셉터가 실행 (예: 인증 확인)
2. 두 번째 인터셉터가 실행 (예: 로깅)
3. 핸들러(컨트롤러)가 실행 (예: 비즈니스 로직 처리)
4. 세 번째 인터셉터가 실행 (예: 응답 포맷 처리)
5. 두 번째 인터셉터에서 후처리 (예: 로깅 후 처리)
6. 첫 번째 인터셉터에서 후처리 (예: 예외 처리)
```

## 비동기 처리
비동기 처리 환경에서는 비동기 처리를 위한 인터셉터 AsyncHandlerInterceptor 인터페이스를 구현해야 한다.  

요청을 처리하는 핸들러가 별도의 스레드에서 실행되는데, 이 스레드는 요청 처리를 담당하는 동시에 메인 스레드는 요청을 완료하지 않고 빠르게 반환될 가능성이 높다. 
따라서 비동기 요청에서는 후처리 단계인 렌더링(view rendering)이나 (postHandle, afterCompletion) 인터셉터가 즉시 호출되지 않고, 
비동기 처리가 완료된 후에 요청이 다시 반환되어 모델을 렌더링하고 최종적으로 응답을 생성하는 과정이 계속 처리된다. 
이때, 비동기 요청의 후처리 단계인 postHandle과 afterCompletion 메서드들이 호출된다. 즉, 비동기 요청이 완료된 후 이 메서드들이 호출되어 후처리가 이루어지게 된다.
  
**API**: [org.springframework.web.servlet.AsyncHandlerInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/AsyncHandlerInterceptor.html)  
<br>

# 참고
* [Interface HandlerInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html)
* [Introduction to Spring MVC HandlerInterceptor](https://www.baeldung.com/spring-mvc-handlerinterceptor)
* [Interceptor pattern](https://en.wikipedia.org/wiki/Interceptor_pattern)