---
title: 8 - Access Token과 Refresh Token 동작 흐름
categories: ci-cd-auth-project
---

# 개요
이전 편에 이어서 Access Token과 Refresh Token의 동작 Flow와 Token 탈취 대응, 로그아웃에 대해서 정리하려고 한다.  

## Access Token과 Refresh Token의 동작 흐름
Access Token과 Refresh Token은 서로 상호작용하는 관계다. 때문에, 이 흐름에 대해서 이해를 하고 있어야 한다.  

1. **사용자 인증과 두 토큰 생성**  
우선 서버에게 사용자가 인증을 요청하면 여러 가지의 Spring Security Filter를 거쳐서 DB의 사용자 정보를 바탕으로 인증 토큰을 발행한다. 바로 이때 발행 목록에는 Access Token과 Refresh Token가 들어가 있고 Access는 몇 분 정도의 매우 짧은 만료 기간이 설정하게 되고, 그리고 Refresh는 상대적으로 한 2주 정도의 기간을 적용받게 된다. 그래서, 이 두 토큰을 사용해서 이후의 인증/인가/보안 절차를 서버는 수행하게 된다.  

2. **AccessToken 요청과 만료**  
인증 이후에는 사용자는 보유한 AccessToken을 요청 헤더에 실어서 서비스를 사용하게 된다. 이때 서버는 AccessToken의 만료 기간을 확인하게 되는데, 만약 만료가 됐다면? 추가적으로 Refresh Token을 요청해서 Access Token을 재발급 할 수 있게끔 한다. 그리고 Refresh Token도 만료 기간을 확인한다. 두 토큰이 만료가 됐다고 가정하면 어떻게 될까? 서버는 이런 경우에는 로그아웃을 시켜서 재로그인을 유도하게 된다.  

이쯤 되면 Access와 Refresh 만료 상황에 대한 여러 경우의 수가 있을 거고 어떻게 고려해서 해결해야할지 고민이 생길 것이라고 생각이 든다.  

### 4개 경우의 수
* **Access & Refresh**  
    * 두 토큰은 만료가 되지 않은 경우다. 따라서 클라이언트의 요청을 정상적으로 처리할 수 있다.  

* **!Access & Refresh**  
    * Access가 만료된 상황이다. 위에 설명하였으므로 넘어가겠다.  

* **Access & !Refresh**  
    * Refresh는 Access 발급 목적으로 사용되는 토큰이다. 보통은 이런 경우가 흔한 상황은 아닐 것이라고 짐작된다. 만약에, 이런 상황이라면 Access 토큰의 만료 기간을 확인 후에 Refresh를 발급하면 된다.  

* **!Access & !Refresh**  
    * 둘 다 만료된 상황이다. 말했듯이 로그아웃 처리하면 된다.  

<br>

## 보안 개발과 Token 탈취 대응 전략(Refresh Token Rotation)
계획대로 흘러가면 정말 최고겠지만 아쉽게도 보안이 들어간 이상 인생처럼 쉽게 흘러가지 않는다.  

보안은 창과 방패와 같다. 그리고 인증/인가 작업은 오류가 발생할 수 밖에 없다. 때문에, 어떤 방법으로 보안을 더 강화할지, 왜 이 방법을 선택했는지 끊임없이 고민하고 그 이유를 찾아야만 한다. 또한, 사용자 편의성도 고려해야 하기 때문에 정말 쉽지가 않다. 즉, 편의성과 보안성 간의 균형을 고려한 기능을 구현해야 하는 것이다.  

인증 과정에서 가장 골치 아픈 건 바로 토큰 탈취다. 각 Token을 해커가 탈취했을 때 어떻게 대응해야 할지 말이다.  

* **Access 토큰 탈취 시**  
    * 이 경우에는 서버가 대응할 방법이 없다. 그저 만료될 때까지 기다리는 수 밖에, 하지만 겨우 몇 분 밖에 되지 않기 때문에 Access Token 탈취는 크게 걱정할 요소가 아니다.  

* **Refresh 토큰 탈취**  
    * Refresh 토큰이 탈취될 경우에는 문제가 심각하다. 왜냐하면 만료 기간이 많이 남았을 가능성이 높기 때문에 이를 탈취할 수 있기 때문이다. 따라서 이를 보완하기 위한 대응책이 바로 RTR(Refresh Token Rotation)다. ROT는 새 AccessToken을 발급할 때 기존 Refresh Token은 만료와 상관없이 새 Refresh Token을 같이 발급받는 것이다.  

나는 이전에 보안은 창과 방패라고 이야기했고, Refresh 탈취는 문제가 심각하다고 주장했다. 즉, ROT 방식에 문제점은 되로 더 큰 문제점을 야기한다. 그 이유는 
새 Refresh Token을 발급하기 이전에 Refresh Token을 탈취해서 새 Access Token을 발급받을 수 있기 때문이다. (이를 Replay Attack라고 한다.) 그래서 이를 막아줄 더 방어력이 단단한 방패가 바로 Blacklist 보안 기능이다.  

* **Replay Attack(재전송 공격)**  
  * 이전에 기록된 쿠키 및 세션 정보를 가로채서 악의적으로 재전송하는 네트워크 공격의 한 형태를 말한다.

### Blacklist 보안 기능 추가와 Redis 저장 방식 도입
Blacklist는 사용자가 Access 만료로 인해 토큰 재발급을 요청할 때 기존의 Refresh Token을 Blacklist에 등록하고, 이후 해당 토큰으로 재발급 시도가 이루어지면 이를 차단하거나 감지하여, 추가적인 대응 전략을 수립할 수 있도록 돕는 역할을 수행한다.  

하지만, 이 방법 역시 허점이 있는데 Refresh Token을 탈취 후 해커가 먼저 선수쳐서 재발급을 받는 것이다. 이렇게 되면 서버는 사용자를 식별할 수 없기 때문에 억울하게 사용자는 쫓겨나게 되고 해커가 계속해서 악용할 수 있게 되는 것이다.  

또 하지만, 해결 방법은 나오기 마련이다. 방금 전에 Blacklist는 감지 기능이 들어가 있다고 했다. 이를 활용해서 토큰을 모두 무효화 전략을 사용하면 된다.  

### 무효화 전략
<img src="/assets/image/cicd-auth/2025-04-07-access token and refresh token flow.png" alt="empty" style="width: 100%; height: auto;">  
- [출처: [인증인가] Access Token, Refresh Token의 저장 위치에 대한 고찰](https://olrlobt.tistory.com/98#access-token-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9D%B8%EC%A6%9D-%EB%B0%A9%EC%8B%9D)  

무효화 전략이란 해커가 먼저 발급 시도로 사용자가 쫓겨나게 되는 억울한 상황을 만들지 않기 위해서 Blacklist에 감지된 토큰과 이후 발급된 토큰까지 모두 무효화 처리하는 방식이다.  

이 방식은 공격 감지는 물론이고 사용자와 해커 모두 무효화 처리되기 때문에, 사실상 가장 괜찮은 수준의 보안 방법이라고 생각한다.  

<br>

## 로그아웃 처리
로그아웃 요청 시 사용 중인 Refresh Token이 아직 만료되지 않았다면 Redis 저장소에서 제거하고, 그리고 Blacklist에 추가해서 혹시 모를 Replay Attack 공격에 대비한다.

<br>

## 참고
- [[인증인가] Access Token, Refresh Token의 저장 위치에 대한 고찰](https://olrlobt.tistory.com/98#access-token%EA%B3%BC%C2%A0refresh-token%EC%9D%98%C2%A0%EB%A1%9C%EA%B7%B8%EC%95%84%EC%9B%83-%EB%B0%A9%EC%8B%9D)
- [Refresh Token Rotation 과 Redis로 토큰 탈취 시나리오 대응](https://junior-datalist.tistory.com/352)