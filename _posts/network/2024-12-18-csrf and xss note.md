---
title: CSRF, XSS 정리
categories: network
---

# 개요
웹 애플리케이션의 보안 취약점을 이용한 공격 방식으로 익히 알려진 CSRF, XSS  
사이트 이용에 있어 신뢰성과 사용자 개인정보 보호 차원에서 개념 및 방어 대응에 대해 학습할 필요가 있다.  

# CSRF(Cross-Site-Request-Forgery)
사이트 간 요청 위조라고 불리며 사용자 권한을 이용하여 의지와 무관하게 공격자가 의도한 행위를 웹 애플리케이션에 요청하게 만드는 공격이다.  

## 공격 방식
1. 악의적 행위 목적으로 csrf 스크립트를 포함한 의도적 페이지 작성
2. 로그인 상태인 사용자에게 그 페이지를 열람 유도 및 열람 시 악위 스크립트 발동
3. csrf 스크립트 실행으로 로그인된 사용자 권한 도용을 이용하여 악의적 행위를 사용자 허가없이 실행

## 공격 예시
```
-이전 양식-
<form method="post"
	action="/transfer">
<input type="text"
	name="amount"/>
<input type="text"
	name="routingNumber"/>
<input type="text"
	name="account"/>
<input type="submit"
	value="Transfer"/>
</form>
```
```
-HTTP 요청 전송-
POST /transfer HTTP/1.1
Host: bank.example.com
Cookie: JSESSIONID=randomid
Content-Type: application/x-www-form-urlencoded

amount=100.00&routingNumber=1234&account=9876
```
```
-악의적 전송 양식-
<form method="post"
	action="https://bank.example.com/transfer">
<input type="hidden"
	name="amount"
	value="100.00"/>
<input type="hidden"
	name="routingNumber"
	value="evilsRoutingNumber"/>
<input type="hidden"
	name="account"
	value="evilsAccountNumber"/>
<input type="submit"
	value="Win Money!"/>
</form>
```
**- docs.spring.io [CSRF 공격이란?] 요약 -**  
특정 신뢰하는 은행 사이트에 로그인한 유저 A는 로그아웃하지 않고 공격자가 악의적 코드를 심어둔 페이지에 접속 후 자신도 모르게 심어둔 코드가 발동되어 공격자에게 100달러가 송금되는 불상사가 발생. 더욱이 공격자는 이런 행위를 JS를 사용해서 자동화가 가능하다는 점이다.  

## 대응 방법
### - docs.spring.io [CSRF 공격으로부터 보호] -
CSRF 공격이 가능한 이유는 피해자 웹사이트의 HTTP 요청과 공격자 웹사이트의 요청이 정확히 동일하기 때문입니다. 즉, 악의적인 웹사이트에서 오는 요청을 거부하고 은행 웹사이트에서 오는 요청만 허용할 방법이 없습니다. CSRF 공격으로부터 보호하려면 악의적인 사이트에서 제공할 수 없는 것이 요청에 있는지 확인해야 두 요청을 구별할 수 있습니다.  
  
**읽기 전용 HTTP 메서드는 CSRF 보호가 가능하다.**  
<ins>HTTP METHOD GET, HEAD, OPTIONS, 및 TRACE</ins>는 읽기만 허용하기 때문에 상태 변경이 되지 않아 안전하고, 무엇보다도 Header에 CSRF Token이 노출되어서는 안된다. 따라서 Spring이 제공하는 CSRF 보안 방법에서 얽매이지 않는다.  

**Synchronizer Token Pattern**  
동기화 토큰 패턴인 CSRF 토큰은 서버 측에서 생성되어야 하며 사용자 세션 또는 요청당 한 번만 생성된다. 공격자가 도난한 토큰을 악용할 수 있는 시간 범위가 요청당 토큰의 경우 최소이기 때문에 세션당 토큰보다 더 안전하다.  
토큰 구성은 다음과 같다.
```
   * 사용자 세션별로 고유
   * 암호
   * 예측 불가능( 안전한 방법을 통해 생성된 큰 난수 값 )  
```
<br>
유저 권한을 악용해서 공격 방법이고, 서버는 HTTP 요청에 대한 식별 분간 능력이 없다. 이에 Spring은 두 가지 보안 방법을 제공한다.  

* **Security Token**  
사용자 요청에 정당성을 검증하기 위해 CSRF Synchronizer Token Pattern을 사용하여 서버가 사용자 요청인지 분간할 수 있다. 
사용자가 HTTP 요청한 페이지에 사용자의 세션 및 CSRF TOKEN 랜덤값을 생성해서 넣어두고, 추후 요청에 해당 랜덤값이 원본과 일치하는지 검사해서 일치할 경우에만 요청에 대한 처리를 수행하도록 한다. CSRF 토큰은 예측 불가능한 랜덤값이기 때문에 공격자가 미리 값을 넣어둘 수 없다.
이를 사용하면 CSRF 공격이 이루어지기 위한 조건의 2번 예측 불가능한 요청 파라미터가 존재하게 되므로 CSRF 공격을 어느정도 방지할 수 있다.  
만약 CSRF 공격을 받는 곳에서 XSS 취약점이 있다면, 공격자의 스크립트를 실행시킬 수 있기 때문에 서버에 요청을 보내 CSRF 토큰을 미리 받아와 공격 코드에 포함시킬 수 있다.  

* **SameSite 쿠키**  
쿠키에 SameSite 옵션을 추가하여 같은 사이트로의 요청이 아닐 때 쿠키를 요청에 포함시킬지 시키지 않을 지 지정이 가능하며 또한, SameSite 속성을 잘 이용하면 CSRF를 효율적으로 방어할 수 있다.  
    * **Lax**  
        동일한 사이트 내의 요청 및 다른 사이트의 GET 요청으로 전송.
        도메인 간 GET 요청으로는 전송되지 않음.  
        예외적으로 form 태그를 이용한 POST 요청에서는 cross-site일 경우에도 쿠키가 전송된다.  
    * **Strict**  
        쿠키가 동일한 사이트 내의 요청으로만 전송됨.  
    * **None**  
        SameSite 옵션을 사용하지 않음(https에서만 적용됨).  

* **Referer 검증**  
요청의 Referer 헤더는 웹 요청시 현재 요청이 발생한 페이지의 URL을 담아서 보낸다.
서버에서는 Referer 헤더를 검사하여 지정한 헤더에서 왔는지, 또는 호스트와 Referer 값이 일치하는지 비교하여 조건을 만족하지 않으면 요청을 거절하는 방식으로 방어할 수 있다. 해당 방법은 XSS 취약점이 존재한다면, Referer 헤더를 변조할 수 있다.  

* **CAPTCHA**  
* **2차인증**  

# XSS(Cross-Site Scripting)
사이트 간 스크립팅라고 하며 공격자가 웹 페이지에 악의적인 스크립트를 삽입하여 사용자의 정보를 탈취하거나 조작하는 공격이다.  

## 공격 방식
1. 사이트의 취약점을 찾아 xss 공격 (게시판이나 웹 메일 등에 스크립트 코드를 삽입)
2. 사용자 웹 페이지 방문
3. 스크립트 실행으로 인한 사용자 쿠키나 세션 탈취나 웹페이지 변조, 악성코드 작동

## 대응 방법
* **HttpOnly Cookie**  
LocalStorage와 일반적인 쿠키는 스크립트를 통해 값에 접근 가능. 따라서 민감한 정보를 클라이언트에 보관하지 않고 HttpOnly 옵션을 사용하여 스크립트로 쿠키에 접근할 수 없게 방지, HttpOnly 쿠키를 통해 SessionId, Token 등이 탈취 당하는 것을 방지할 수 있음. 

* **입출력 값 검증과 필터링**  
입력 받은 값을 제대로 검사하지 않고 사용할 경우 발생하며, 결과로 사용자는 의도치 않은 동작을 수행하거나 쿠키, 세션 등의 정보를 탈취  

* **CSP(Content Security Policy)**  
웹 페이지에서 실행될 수 있는 스크립트의 출처를 제한함으로써 XSS 공격을 방지  

* **XSS 특수문자 치환**  

# 비교
두 공격 방식 모두 **웹 사이트 취약점 공격 방법으로 공통점**을 가지고 있지만, 특성을 비교했을 때 차이점이 명확하게 드러난다.

## 차이점

```
[종류]      [사용자 상태]   [공격 대상]     [목적]                  [관점]
CSRF        로그인          서버            사용자 권한 도용        사이트가 사용자를 신뢰
XSS         비로그인        클라이언트      쿠키, 세션 등 탈취      사용자가 사이트를 신뢰
```

# 참고
* [크로스 사이트 요청 위조(CSRF)](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html)  
* [XSS와 CSRF 차이점 및 대응 방안](https://velog.io/@haizel/XSS%EC%99%80CSRF-%EC%B0%A8%EC%9D%B4%EC%A0%90-%EB%B0%8F-%EB%8C%80%EC%9D%91-%EB%B0%A9%EC%95%88)  
* [보안취약점 XSS와 CSRF](https://toma0912.tistory.com/101)  
* [[Security] XSS와 CSRF 공격](https://yoo-dev.tistory.com/19)  
* [웹 보안의 기본: CSRF와 XSS 공격 이해하기](https://f-lab.kr/insight/understanding-csrf-and-xss-attacks)  