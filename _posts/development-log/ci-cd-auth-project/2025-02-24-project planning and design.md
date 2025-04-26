---
title: 1 - 프로젝트 기획 및 설계
categories: ci-cd-auth-project
---

# 기획 및 설계
## CI/CD와 Auth Login
프로젝트 주제는 GitHubActions를 활용한 CI/CD와 SpringSecurity와 JWT를 결합한 인증 시스템이다.

CI/CD는 개발 단계부터 배포까지 모든 과정을 자동화하는 중요한 역할을 하기 때문에 당연히 필요한 부분이라고 생각했고, 아직 공식적 프로젝트에 녹여낸 적이 없기 때문에 이 프로젝트에 넣기로 마음을 먹게 되었다.  
GitHubActions는 Jenkins와 항상 비교되는데, 이전에 GithubActions으로 CI/CD를 구축한 경험이 있어 이번에는 더 나은 구현을 할 자신이 있다. 무엇보다 빠르게 구현한 후 다른 기능을 탐색하는 것이 경험상 효율적이라고 판단해 GitHub Actions를 선택했다.  

이전까지 나는 프로젝트에 Session 로그인 기능 구현 경험이 있다. 또한, Spring Handler와 Filter 그리고 일부 인증 방식을 학습해왔기 때문에 JWT Token 기능을 꼭 한 번 완성해보고 싶었다.  

## 프로젝트 설계
### 기술 스택
![Image](https://github.com/user-attachments/assets/db6532a0-df7d-4008-9f00-4f0641f3d6aa)   

* Java 17 + SpringBoot 3 버전으로 서버를 개발하려고 한다.
* Spring SecuJWT + RESTFul API + Swagger or Postman
<br>

### ERD
<img src="https://github.com/user-attachments/assets/f6a1915e-1269-4967-b096-920ecd3f1459" style="width: 50%; height: auto">  

클라이언트는 액세스 토큰을 가지고 있고, 서버는 리프레시 토큰을 보유하는 구조이기 때문에 두 가지 테이블을 설계하였다. 현재는 임시 설계이므로, JWT 방식에 대해 학습한 후 계속해서 변경될 예정이다.
<br>

### 규칙
**branch rule**  
![Image](https://github.com/user-attachments/assets/94a0b4db-b227-40f6-8a75-8619da68b0a3)  

**commit rule**  
![Image](https://github.com/user-attachments/assets/bddb5750-65f0-4e28-979b-616897da43e4)  

GitHub Flow 전략을 선택하게 되었다.  
이 전략을 선택한 이유는 브랜치 구조와 규칙이 직관적이고 간단하여 소규모 개인 사이드 프로젝트에 적합하다. 또한, PR 방식의 자동화 시스템이 release 브랜치 역할을 대체할 수 있어, CI/CD를 활용한 자동화된 배포와 결합하면 더 유연하고 효율적인 개발이 가능하다.  


<!-- 개발일지란 내가 배운 것을 정리하고, 그에 대한 간단한 소감, 고민들을 기록하는 것입니다
오늘 혹은 이번주에 배운 것에 대해 다시 한번 복습 겸 정리하고, 발생한 오류를 어떻게 해결했는지 등을 기록
어떤 고민들이 있었는지와 간단한 소감을 함께 적어두는 개발을 위한 일기 -->

<!-- 1. 어떤 어려움을 겪었고 어떻게 해결했는지
어떤 기술을 무엇때문에 썼는지 구체적으로
2. 기승전결
3. 상황 -> 과제 -> 행동 -> 결과 수치화 -->



