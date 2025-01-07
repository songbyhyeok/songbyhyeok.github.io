---
title: MVC DispatcherServlet을 통한 JSON 역직렬화 과정
categories: spring
---

# 개요
MVC 아키텍처와 직렬화/역직렬화 과정을 기반으로, 객체 변환 및 생성의 동작을 학습한다.

# MVC 구조와 동작 처리 과정 분석
Spring MVC Pattern의 구조와 동작 처리 과정을 분석하고, 직렬/역직렬화 변환을 통해 데이터가 생성될 수 있는지 살펴봄으로써 "null"이 발생한 원인을 분석할 수 있는 토대를 마련하고자 한다.

## 필드 매핑은 어떻게 처리되는 것인가?  
@RequestBody는 비동기 통신으로 요청된 Http 본문 body의 포맷 데이터에 따라 적절하게 역직렬화 처리를 목적으로 Controller의 파라미터 메소드에 Http 본문을 전달한다. 이는 곧, HttpMessageConverter가 처리하게 되는데, 더 내부적으로는 Object Mapper가 이를 수행하게 된다. 하지만 해당 정보만으로는 원인 파악을 할 수 없기 때문에 내부적으로 구조 형식이 어떻게 처리가 되는지 접근해야 한다.

**-개념 파악-**  
기록이 필요한 객체 유형의 속성을 외부 형식으로 저장했다가, 필요 시 이를 다시 복구하는 기술을 뜻한다.  

**직렬화**  
객체(필드)의 속성과 데이터를 파일화하여 외부에 저장한다. 직렬화 가능한 클래스들은 기본 생성자가 default여야 하다.  

**역직렬화**  
직렬화로 저장된 외부 파일을 다시 이전 객체로 만드는 것, 생성자를 거치지 않고 **리플렉션**을 통해 객체를 구성한다.

## DispatcherServlet 설계
![image](https://github.com/user-attachments/assets/adcf4a02-01e3-40db-ae71-97040f66644c)  
**출처: Spring MVC Part.1**  

MVC, RESTful 패턴에서의 중앙 배차 시스템의 기점이다. Front Controller 디자인 패턴에 따라 Http 요청을 받아서 원활한 처리를 위해 역할 분배 및 위임한다. 
즉, DispatcherServlet은 요청에 따라 적절한 컨트롤러를 찾고, 내부 처리 메서드를 호출시켜 비즈니스 로직을 수행한다. 바로 여기까지의 구조 및 동장 방식이 @RequestBody와 관련되어 있다.  

### Dispatcher 컨트롤러 처리 설계
![image](https://github.com/user-attachments/assets/87603a26-43eb-4c9c-8bc1-8ad844f01185)

### HandlerMapping(핸들러 결정)
```
@PostMapping("/signup/verification/phones/auth")
    public ResponseEntity<Boolean> generatePhoneAuthCode(HttpSession session, HttpServletRequest request, @RequestBody SmsAuthRequest smsAuthRequest) {


        System.out.println(smsAuthRequest);
        final DefaultCsrfToken csrfToken = (DefaultCsrfToken) session.getAttribute(
                "CSRF_TOKEN");

        assert(csrfToken == request.getAttribute(csrfToken.getParameterName()));

        return ResponseEntity.ok()
                .header(csrfToken.getHeaderName(), csrfToken.getToken())
                .body(memberService.requestPhoneAuth(smsAuthRequest));
    }
```

웹 애플리케이션의 다양한 요청을, Handler Mapping은 어떤 Handler(Controller)가 처리할지 결정하는데, 주로 요청에 관한(URL, 메서드, 헤더) 등 기반으로 적절한 핸들러를 선택하고 매핑한다. 그리고 일반적으로 HandlerMapping을 상속하는 RequestMappingHandlerMapping을 통해 동작된다.

### HandlerAdapter(컨트롤러의 메서드 호출 처리)
@RequestMapping을 처리하는 핸들러, 유연한 방식으로 Http 요청 처리를 할 수 있는 interface이다. 메소드를 특정 URL, 파라미터에 매핑하는 ArgumentResolver 함께 사용된다.  

**Dispatch Controller 동작 정리**  
1. 클라이언트 요청이 DispatcherServlet에 전달된다.   
2. HandlerMapping을 통해 요구한 컨트롤러를 참조한다.  
3. 됐을 경우, DispatcherServlet은 HandlerAdapter에게 컨트롤러의 메서드 호출 처리를 위임한다.  

## Handler 구체적 처리 설계
![image](https://github.com/user-attachments/assets/a9f4ba2e-6812-409a-8020-7f2327890dd2)

### Request Mapping(핸들러에게 요청 전달)
Handler에 어떤 방식으로 처리할지 정의를 하는데, 이때 들어온 요청을 핸들러와 매핑하기 위해 사용되는 기술이다.  
해당 어노테이션을 컨트롤러 클래스 레벨에 적용하면 해당 컨트롤러의 모든 메서드에 적용되며, 메서드 레벨에 적용하면 해당 메서드만 특정 URL에 매핑된다. 그리고 
Controller 위에 @Mapping 을 명시하면 이를, 확인하고 RequestMappingHandlerMapping을 생성시켜 DispatcherServlet에 등록된다.

### ArgumentResolver(메서드 파라미터 결정)
실제 객체명은 HandlerMethodArgumentResolver이며, ArgumentResolver는 @RequestMapping과 HandlerAdapter 사이에서 상호작용하는데, HandlerAdapter와는 Controller가 필요로 하는 다양한 파라미터의 값(객체)을 생성시켜 이를, 반환해 주면 Adapter가 Controller를 호출하면서 값을 넘긴다.

### ReturnValueHandler
정확한 명은 HandlerMethodReturnValueHandler이며, Controller에서 반환된 객체를 Http 응답으로 변환하는 역할(@ModelAndView, @ResponseBody....)

**동작 이해** 
1. HandlerAdapter가 ArgumentResolver에게 Controller에 필요한 파라미터를 요청한다.
2. ArgumentResolver는 supportsParameter() 호출해서 해당 파라미터를 사용할 수 있는지 확인. 됐을 경우 resolveArgument()를 호출해서 HttpMessageConverter에게 데이터를 직렬/역직렬화를 위임한다.
3. HandlerAdapter는 반환받은 파라미터를 바탕으로 Controller를 처리한다.

## HttpMessageConverter에 의한 요청/응답
Http 응답에 있는 Message Body를 읽어 이를, 요청/응답 간의 변환 작업을 처리한다. HttpMessageConverter는 미디어 타입(Content-Type),  
class Type 두 개를 체크해서 사용/생성 객체 여부를 결정한다. 

**미디어 타입과 클래스 타입?**  
인터넷에서 전송되는 문서의 형식을 **미디어 타입(Multipurpose Internet Mail Extensions)**라고 하며,  
직렬 및 역직렬화에 의해 생성되는 객체 타입을 **클래스 타입**이라고 한다.  

### Converter 생성 및 동작
**BackgroundPreinitializer**  
-> MessageConverter를 생성 및 실행  
**MessageConverter**  
-> AllEncompassingFormHttpMessageConverter(), Converter 종류를 생성  

### 대표적 Converter
**ByteArrayHttpMessageConverter (order 0)**  
-> 클래스 타입은 byte[]이며 미디어 타입은 */* 이다.  

**StringHttpMessageConverter -> (order 1)**  
-> 클래스 타입은 String이며 미디어 타입은 */* 이다.  

**MappingJackson2HttpMessageConverter (order 2)**  
-> 클래스 타입은 객체 or HashMap, 미디어 타입은 application/json이다.  
-> ObjectMapper를 사용하여 실질적인 역직렬화를 수행한다.  

각 Converter는 우선순위가 적용되어 있으며, 클래스 타입과 미디어 타입에 따라 객체 생성이 결정된다.      

### Convert 종류에 따른 코드 예시
**--ByteArrayHttpMessageConverter--**  
```
content-type : application/json

@RequestMapping
void hello(@RequestBody byte[] data) {}
```

**--StringHttpMessageConverter--**  
```
content-type : application/json

@RequestMapping
void hello(@RequestBody String data) {}
```

**--MappingJackson2HttpMessageConverter--**  
```
content-type : application/json

@RequestMapping
void hello(@RequestBody Object data) {}
```

올바른 Converter를 찾지 못한다면 관련된 예외가 발생하게 된다. 때문에 content-type과 classType이 조건에 맞는지 확인해야 한다.

### 요청(can)과 응답(read) 
공통적으로 canRead or canWrite를 호출해서 Converter가 메시지(classType, MediaType)를 읽거나 작성할 수 있는지 확인한다.  
만약 조건 수행이 가능하다면, read or write 메서드를 통해 읽고 쓰는 즉, 객체 생성 반환 동작을 수행한다. 

**요청(can)**  
**ArgumentResolver** 요청에 대한 다양한 타입의 인자 값 변환, 필요 객체를 생성한다.  
requestResponseBodyMethodProcessor 객체에서 수행되며 우선순위에 따라, canRead()로 클래스 타입, 미디어 타입이 일치하는지를, Converter가 읽어 올 수 있는지 확인한다. true일 경우 read() 호출해서 객체 생성 및 반환하며, 
false일 경우 우선순위에 따라 다음 객체를 확인한다.   

**응답(read)**   
**ReturnValueHandler** 응답에 대한 다양한 타입의 인자 값 변환을 제공한다.  
우선순위에 따라, canWrite()로 클래스 타입, 미디어 타입이 일치하는지를, Converter가 작성할 수 있는지 확인한다. true일 경우 write() 호출해서 응답 메시지 바디에 데이터 생성을 하며, false일 경우 다음 우선순위로 넘어가서 다시 확인한다.  
 
요청은 Content-Type의 미디어 타입을 확인했다면, 응답은 Accept의 미디어 타입을 확인한다.

## 실질적인 객체 생성은 ObjectMapper
Jackson 라이브러리에서 제공하는 객체로서, 직렬/역직렬화 기능을 제공한다.  
Java 객체를 Json 데이터 직렬화는 writeValue()를 호출하고, Json 데이터를 Java 객체로 변환인 역직렬화는 readValue()를 호출한다.

### Jackson2Converter을 통해 생성되는 ObjectMapper
HttpMessageConverter는 포맷 방식에 따라 적절한 구현체를 호출하게 된다. 이 중, Json 포맷 형식으로 요청이 들어오면 MappingJackson2HttpMessageConverter 구현체를 통해 ObjectMapper를 사용하여 직렬/역직렬화를 수행한다.

### ObjectMapper가 Java 객체 생성을 하기까지
ObjectMapper는 기본 생성자를 통해 DTO를 생성하고, Reflection을 통해 필드 정보를 확인하여 필요한 데이터를 할당하는 방식으로 자바 객체를 형성한다. 
만약, 기본 생성자가 없는 경우 deserializeFromObjectUsingNonDefault 라는 메서드를 호출하여 이에 대응하고자 하지만, 클래스 정보를 파악할 수 있는 특정 조건에 해당되지 않는 경우, 자바 객체를 매핑하는 것을 실패할 수 있으므로, @RequestBody에 해당하는 DTO는 일반적으로 기본 생성자가 있는 것이 좋다.  

**-개념 파악-**  
**Jackson**   
Json, XML, YAML, CSV 등 다양한 포맷 확장자를 지원 및 데이터 처리를 지원하는 라이브러리이다. 전송 방식은 스트림 방식을 사용하기에 빠르고, 다양한 매핑 방식을 제공하기 때문에 유연하다. 
매핑 방법에는 프로퍼티 (g.s)etter 방식, @JsonProperty, @JsonIgnore, @JsonAutoDetect 등이 있다.  

**Reflection**  
구체적인 Class Type을 알지 못하더라도 해당 Class의 method, type, variable들에 접근할 수 있도록 해주는 자바 API이며, 컴파일된 바이트 코드를 통해 
Runtime에 동적으로 특정 Class의 정보를 추출할 수 있는 프로그래밍 기법이다. 투영, 반사 라는 사전적인 의미를 가지고 있다.

## Json 역직렬화 동작 과정 정리
Client 요청 -> DispatcherServlet -> RequestMapping -> HandlerMapping -> DispatcherServlet -> HandlerAdapter -> ArgumentResolver -> HttpMessageConverter ->
canRead → contentType(json) → MappingJackson2HttpMessageConverter → ArgumentResolver → read → ObjectMapper → Reflection -> Java Object(DTO) -> ArgumentResolver -> HandlerAdapter

# 참고
* [@RequestBody vs @ModelAttribute](https://tecoble.techcourse.co.kr/post/2021-05-11-requestbody-modelattribute/)
* [Reflection API 간단히 알아보자.](https://tecoble.techcourse.co.kr/post/2020-07-16-reflection-api/)  
* [[Spring] @RequestBody - Spring의 JSON-Java Object 변환 원리](https://velog.io/@beberiche/Spring-RequestBody-Spring-Object-Mapping-%EC%9B%90%EB%A6%AC)  
* [[Spring] Jackson 라이브러리 이해하기.](https://mommoo.tistory.com/83)  
* [직렬화와 역직렬화 (Serializable)](https://codevang.tistory.com/164)  
* [[Spring MVC] @RequestBody 동작 원리 [1] - Http Message Converter](https://gist.github.com/taekwon-dev/666eefdddda2207b86368dcc5ab12a4b)  
* [[Spring] HttpMessageConverter가 적용되는 시점에 대해](https://bepoz-study-diary.tistory.com/374)  
* [HTTP 메시지를 spring에서 사용하기 쉽게 만들어 주는 MessageConverter](https://velog.io/@gloom/HTTP-%EB%A9%94%EC%8B%9C%EC%A7%80%EB%A5%BC-spring%EC%97%90%EC%84%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-%EC%89%BD%EA%B2%8C-%EB%A7%8C%EB%93%A4%EC%96%B4-%EC%A3%BC%EB%8A%94-MessageConverter)
* [15)HTTP 메시지 컨버터 , RequestMappingHandlerAdapter 구조](https://keeeeeepgoing.tistory.com/183)  
* [@RequestBody는 어떻게 바인딩 되는걸까 with 디버깅 과정](https://strong-park.tistory.com/entry/RequestBody%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EB%B0%94%EC%9D%B8%EB%94%A9-%EB%90%98%EB%8A%94%EA%B1%B8%EA%B9%8C-with-%EB%94%94%EB%B2%84%EA%B9%85-%EA%B3%BC%EC%A0%95)  
* [https://blogshine.tistory.com/445](https://blogshine.tistory.com/445)  
* [[Spring] HTTP Request 를 처리하는 과정 - DispatcherServlet 원리](https://ibocon.tistory.com/208)
* [HandlerMapping과 HandlerAdapter는 왜 나뉘었나요?](https://stonehee99.tistory.com/24)
* [Handler Mapping과 Request Mapping이란?](https://sharonprogress.tistory.com/196)
* [[Spring] HandlerMethodArgumentResolver 동작 원리](https://kookiencream.tistory.com/120)
* [HttpMessageConverter와 직렬화, 역직렬화](https://localhost8586.gitbook.io/heo-y-y/catalogue/framework/spring/undefined-2)
* [[DevYGwan:티스토리]](https://swmobenz.tistory.com/24)