---
title: 포트 포워딩 정리
categories: network
---

## 간략 설명
패킷을 보낼 때 라우터나 방화벽 요소들로 인해서 목적지에 보낼 수 없는 상황일 때, 포트 포워딩 기술을 사용하여 경유지 주소를 통해 우회하여 목적지에 정상적으로 통신할 수 있다. 여기서 경유지 역할로 보편적으로 사용되는 SSH 22 PORT (과거엔 보안 기능이 없는 Telnet 23 PORT 사용)을 사용하고 있는데, 이 녀석을 이용하여 SSH Tunneling을 명령어를 통해 생성하면 하나의 Proxy 서버가 구축이 된다. 

## SSH 터널?
패킷을 주고 받을 수 있는 암호화된 SSH 연결 통로를 의미한다. 외부에서 데이터를 도청하거나 가로챌 수 없어, 기밀성 및 무결성을 보장한다. 하지만 내부 허가된 자가 나쁜 의도로 접근하여 백도어 공격을 통해 MALWARE와 같은 침투를 할 수 있어 주의해야 한다.

## 포워딩 종류
* **로컬 포워딩**  
![image](https://github.com/user-attachments/assets/968c3ddc-76a4-4caa-a7d8-fee1c333f002)  
포워딩 기술 중 보편적으로 사용되는 유형으로서, ssh 터널을 통해 from 로컬에서 to private 목적지까지 우회해서 통신이 가능한 구조  

        ssh -i [.pem] -fN -L [localhost:port]:[목적지:3306] [경유지 이름@경유지 주소]
        ssh -L 80:intra.example.com:80 gw.example.com

* **원격 포워딩**  
![image](https://github.com/user-attachments/assets/13283df5-6e64-49a9-bf84-9cf7aebafc56)
로컬 유형과 반대로 인트래픽이 방화벽에 의해 차단되어 접근 불가한 상태인(인바운드 차단) 원격지 서버가 클라이언트 주소로 터널을 생성시켜 통신을 가능케 하는 방법

        ssh -R 8080:localhost:80 public.example.com

* **동적 포워딩**  
![image](https://github.com/user-attachments/assets/d7581996-547c-48dc-859a-8d07ec00d64a)
로컬과 원격 방식은 통신 관계가 오직 1:1 이지만, 해당 방식은 SOCKS5 프록시 서버를 구축하여 로컬에서 지정한 호스팅 PORT를 통해 원격지의 모든 PORT에 접근할 수 있다.  

  **프록시?, SOCKS?**  
우선 프록시는 흔히 말하는 중계, 경유지 역할을 대신하는 간접적인 서버를 말한다. 여태 포워드 개념에 대해서 다루고 있는데 포워드 매핑 개념도 프록시의 한 종류이다. 동적 포워딩에서 쓰이는 프록시는 사용자가 경유지 서버를 목적으로 하나를 구축하게끔 한다.  
SOCKS는 프록시 서버가 클라와 서버가 패킷을 라우팅 처리 중 사용되는 프로토콜을 말하는데, 여기서 라우터와 비슷한 개념이라 혼동될 수 있지만 라우터는 들어온 패킷을 지정된 경로로 우회시키는 역할을 수행하는 장치로서 OSI 3계층인 네트워크에서 동작하고 SOCKS는 5계층인 세션에 위치한다. 그리고 프록시는 APP 계층에서 수행하게 된다.

        ssh -D localhost:1080 [socksname@socks주소]

## 참고
- [link1](https://omoknooni.tistory.com/m/73)
- [link2](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=alice_k106&logNo=221364560794)
- [link3](https://www.ssh.com/academy/ssh/tunneling-example)
- [link4](https://datawookie.dev/blog/2023/12/ssh-tunnel-dynamic-port-forwarding)
- [link5](https://chamibuddhika.wordpress.com/tag/port-forwarding)