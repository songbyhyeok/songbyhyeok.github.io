---
title: Java with Gradle workflow denied issue 해결하기
categories: github_actions
---

## 빌드 시도 시 denied 문제
![denied issue](https://github.com/user-attachments/assets/0e660572-8982-41a4-ab70-00bb4f24e153)

github actions 상에서 build 과정에서 이미지의 내용에 있는 거부 명령을 받게 되었다.

## 왜 거부를 당했지?
![image](https://github.com/user-attachments/assets/b445ed0b-8db5-4f32-b45c-93bd283723fa)

Gradle 공식문서에 위와 같이 처리를 할 수 있지만 실행 가능하지 않다고 설명이 되어 있고, 해결 명령어를 제시하고 있다.  
[https://docs.gradle.org/current/userguide/troubleshooting.html](https://docs.gradle.org/current/userguide/troubleshooting.html)

## 명령어를 삽입하자.
```yaml
jobs: build: steps: 에서  
- name: Grant execute permission for Gradle
    run: chmod +x ./gradlew
```

저 코드를 삽입하는 것으로 실행 권한을 부여할 수 있게 된다.

```yml
# chmod +x ./gradlew 명령어 해석**  
chmod -> 파일 권한 변경 명령어  
'+x' -> 실행 권한 추가 옵션  
'./gradlew' -> 현재 디렉토리에 있는 gradlew 파일 지정
```