---
title: Redirect, Foward 개념과 차이점
categories: Spring
tag: [SPRING]
---

## 정의
두 용어 모두 클라이언트 요청에 대한 서버의 URL 처리 기술 방식을 말한다.  
**Redirect**  
사용자 요청에 대해 서버는 기존 URL이 아닌 새 URL을 발급한다.  
**Forward**  
단일적 처리 방식이라고 할 수 있으며, 요청에 대한 처리 과정을 사용자는 알 수 없고 첫 URL을 유지하게 된다.

## 간단 예제로 이해하기
```java
@Controller
@EnableAutoConfiguration
public class NavigationController {

    @RequestMapping("/index-page")
    public String index(@PathParam("name") String name, ModelMap modelMap) {
        System.out.println("param name=" + name);
        modelMap.put("name",name);
        return "test";
    }

    @RequestMapping("/redirect")
    public String redirect() {
        return "redirect:index-page";
    }

    @RequestMapping("/forward")
    public String forward() {
        return "forward:index-page";
    }
}

// result!
redirect?name=henry
-> Hello, null!

forward?name=henry
Hello, henry!
```

간단 실험으로 redirect와 forward의 이해와 차이점으로는, redirect는 요청 시 상태값을 보존하지 않지만 forward는 저장하고 있는 것을 알 수 있다.

## 왜? redirect는 새 url로 우회 처리하는 것일까?
**Post/Redirect/Get Pattern**  
![image](https://github.com/user-attachments/assets/2e2cc116-27a8-44b8-864c-0a44ee8b190a)

![image](https://github.com/user-attachments/assets/ad77c9ed-acbf-4ef6-8ef4-5b2b2f07f788)  
PRG 패턴은 redirect를 왜 사용해야 하는가를 납득시킬 수 있게 해준다. 첫 그림과 같이 forward 방식으로 post 처리를 하게될 경우 같은 물건을 또 주문을 하게 되는 불상사가 발생하게 된다. 그러나 두 번째 그림처럼 redirect 방식을 사용하면 주문 건에 대해 처리 후 주문 완료에 대한 페이지 부분 url만 발급시키기 때문에 새로고침 시 중복 건수가 발생할 수 없는 시스템을 만들게 된다.

## 정리
두 개념의 차이점은 다음과 같이 정리할 수 있다.
redirect는 url을 새로 발급 및 상태 값을 저장하지 않고, 반면 forward는 기존 url을 사용하고 상태 또한 저장하여 재사용이 가능하다. 두 개념을 학습 시 각기 목적에 따라 적합한 기술 방식을 채택할 수 있게 해 준다.

## 참고
- [https://doublesprogramming.tistory.com/63](https://doublesprogramming.tistory.com/63)
- [https://www.baeldung.com/servlet-redirect-forward](https://www.baeldung.com/servlet-redirect-forward)
- [https://en.wikipedia.org/wiki/Post/Redirect/Get](https://en.wikipedia.org/wiki/Post/Redirect/Get)
- [http://www.henryxi.com/spring-mvc-forward-redirect-example](http://www.henryxi.com/spring-mvc-forward-redirect-example)
