---
title: 10 - Nginx를 활용한 무중단 배포 구축기
categories: ci-cd-zero-downtime-auth
---

# 무중단 배포
배포 과정에서 서버가 일시 중단되는 문제에 대해 생각해 본적이 있는가?   
만약, 서비스가 중단된다면 잠깐이지만 크게 3가지 정도의 문제점들이 발생할 수 있다.  
1. 실제 운영 환경에서 서비스 중단은 곧 손실
* 배포나 갑작스러운 장애로 서버가 다운될 경우 매출 손실과 더불어 고객 이탈이 발생하게 된다.
2. 새벽 시간 대에 전통적 배포가 이루어진다.
* 운영 효율성을 고려해 새벽 시간 대에 배포 작업을 하게 되는데, 이는 곧 야근과 그리고 장애 발생 시 긴급 대응으로 이어진다.
3. 테스트 및 배포 과정은 생각보다 길다.
* 여러 업무를 처리해야 하지만 배포로 지장이 생긴다면 생산성 저해로 이어지게 된다.

이와 같은 문제는 기업과 사용자 모두에게 손해를 초래할 수밖에 없다.  

## 무중단 배포 방식
무중단 배포 방식은 크게 세 가지 정도 널리 사용되고 있다.
- 롤링(Rolling Update) 방식
- 블루 그린(Blue-Green Deployment) 방식
- 카나리(Canary Release) 방식

내가 채택한 방식은 블루 그린 방식이다. 이 방식은 운영 환경에서 구 버전을 구동하다가 배포 때, 모든 트래픽을 구에서 신 버전으로 전환시키는 마치 Context Switching과 같은 방식이라고 할 수 있다.  

블루 그린 방식 방식은 사용할 수 밖에 없는 이유는 다음과 같다.  
- 신 버전 배포 중에 서버 과부하가 발생할 확률이 구조상 현저히 적다.
- 테스트 중 장애 발생 시 빠른 롤백을 지원한다.  

하지만, 마냥 좋을 수는 없다. 서버를 전환할 때 물리적 서버가 두 개가 필요하므로 현재 사용되는 ec2 free tier 서버에서 비용을 감당하기 버겁다. 그렇지만, 
도커 컨테이너 기술을 사용한다면 이 문제를 해결할 수가 있다.  

## Docker 컨테이너 기술
<img src="/assets/image/projects/ci-cd-zero-downtime-auth/zero-downtime-deployment.png" alt="empty" style="width: 50%; height: auto;">  
도커는 운영체제 위에 도커 엔진을 얹혀서 여러 개의 독립된 컨테이너를 관리하는 가상화 플랫폼이다. 때문에, 각 컨테이너는 필요한 만큼의 리소스만 사용하므로, 보다 최적화된 환경에서 효율적으로 애플리케이션을 구동할 수 있다. 또한, 버전이 다른 동일한 두 EC2 서버라도 도커 이미지를 활용하면 변경 및 삭제 과정을 반복하면서 동일한 환경을 손쉽게 구축할 수 있어, 추가적인 비용 부담이 거의 없다.  

참고로, 여러 서버를 관리할 때 또 하나의 문제는 새 버전의 서버가 기존과 다른 모듈이나 라이브러리 버전 환경에서 제대로 동작하지 않을 수 있다는 점이다. 즉, 버전 차이로 인해 호환성 문제가 발생할 수 있다. 하지만 도커 컨테이너는 애플리케이션 코드, 라이브러리, 런타임, 설정 파일 등 실행에 필요한 모든 요소를 하나의 패키지로 묶어, 완전히 독립된 실행 환경을 제공하기 때문에 이러한 호환성 문제를 효과적으로 해결할 수 있다.  

자 여기까지 구성한다면 배포를 할 수 있는 것일까? 답은 아니오다. 왜냐하면, 아직 서버를 효율적으로 관리하고 제어해 줄 기술이 추가되지 않았기 때문이다.  

## 왜 Nginx인가?
Nginx는 여러 배포 방법 중에서 설치가 간편하고, 경량이며, 설정 변경 시 연결이 끊기지 않는 구조를 제공한다. 더 구체적으로 Nginx를 사용할 수밖에 없는 이유는 두 가지 기술인 Reverse Proxy와 Load Balancing을 사용할 수 있기 때문이다. 바로 이점 때문에 무중단 배포를 구현하기에 매우 적합하다.  

Nginx는 중단 배포 전략 중 하나인 Blue-Green Deployment를 매우 효과적으로 구현할 수 있다. 왜냐하면 요청 트래픽을 리버스 프록시를 통해 가동 중인 서버에 전달하다가, 배포 때 기존 서버에 유입되고 있는 트래픽을 새 서버로 바꾸면 되기 때문이다.  

Blue-Green 전략은 운영 중인 버전(예: Blue)과 신규 버전(예: Green)을 동시에 유지하면서, Nginx 설정을 통해 트래픽을 점진적으로 Green으로 전환하거나, 문제가 있을 경우 다시 Blue로 손쉽게 되돌릴 수 있다. 이러한 구조는 서비스 중단 없이 새로운 버전을 배포하고, 빠르게 롤백할 수 있는 안정적인 배포 환경을 만든다.  

## 구조 및 동작 원리
<img src="/assets/image/projects/ci-cd-zero-downtime-auth/zero-downtime-deployment-2.png" alt="empty" style="width: 50%; height: auto;">  

전체적 구조는 위 그림과 같이 EC2(Ubuntu), Docker, Nginx, SpringBoot로 구성된다.  

EC2는 전체 서버를 의미하며, 해당 인스턴스에는 Ubuntu 운영체제가 설치되어 있다. 그리고 애플리케이션 간의 충돌을 방지하고 독립적인 실행 환경을 제공하기 위해 Docker 엔진이 설치되어 있다.  

Nginx는 웹 서버로서 클라이언트의 HTTP 요청을 받아 Reverse Proxy 방식으로 WAS 서버에 트래픽을 전달한다. 배포 중에는 Nginx가 애플리케이션 계층(Layer 7)에 위치하기 때문에, L7 로드 밸런싱을 통해 기존 트래픽을 새로운 버전의 Spring Boot 컨테이너로 유연하게 전환할 수 있다.  

### Nginx + Blue-Green 전략을 활용한 무중단 배포 동작 원리
1. **초기 상태 (Green 활성 상태)**  
운영 중인 서비스는 Nginx가 Green 환경(Spring Boot 컨테이너 A) 으로 모든 트래픽을 전달하고 있다. 클라이언트는 이 환경을 통해 정상적으로 서비스를 이용하고 있다.

2. **새로운 버전 배포 (Blue 환경 준비)**  
새로운 버전의 애플리케이션을 Blue 환경(Spring Boot 컨테이너 B) 에 배포한다. 이때, 아직 Nginx는 트래픽을 Green에만 전달하고 있으므로 사용자는 변경 사항을 인지하지 못한다.

3. **헬스 체크 및 사전 테스트**  
Blue 환경에서 서비스가 정상적으로 동작하는지 헬스 체크 및 테스트를 수행한다. 오류가 없고 안정성이 확인되면 다음 단계로 넘어간다.

4. **Nginx 설정 변경**  
Nginx 설정을 변경하여 트래픽의 라우팅 대상을 Green → Blue 로 전환한다. 이 변경은 거의 실시간으로 적용되며, 사용자 요청은 중단 없이 Blue 환경으로 전환된다.

5. **롤백 대비**  
만약 Blue 환경에서 문제가 발생하면, Nginx 설정을 다시 Green 환경으로 되돌려 빠르게 롤백할 수 있다. 이를 통해 다운타임 없이 안전한 배포를 수행할 수 있다.

6. **후처리 및 정리**  
Blue 환경이 안정적으로 운영되면, 이전 Green 컨테이너는 종료하거나 보관하여 리소스를 정리한다.

## 무중단 배포 구축하기
### Nginx 설정 파일에서 프록시와 로드밸런싱 설정하기
Nginx의 다음 경로에는 서버 관련 설정 파일이 들어있다. 해당 서버 설정 파일을 열어서 스프링 프로젝트에 연동될 수 있도록 설정을 바꿔보겠다.  
- /etc/nginx/conf.d/default.conf
- /etc/nginx/conf.d/service-env.inc

<img src="/assets/image/projects/ci-cd-zero-downtime-auth/zero-downtime-deployment-3.png" alt="empty" style="width: 50%; height: auto;">  

위 자료는 default.conf 파일 내용 중 일부이다.  

#### proxy_pass
- 클라이언트 요청을 $service_url이라는 변수에 저장된 값으로 전달한다.  

#### proxy_set_header
- Host 헤더 전달: 백엔드 서버가 원래 요청한 호스트 이름을 알 수 있도록 한다.  
- 클라이언트 IP 전달: 백엔드가 클라이언트의 실제 IP 주소를 로그나 인증 등에 사용할 수 있도록 한다.  
- 인증 헤더 전달: 인증 토큰 등을 백엔드로 넘길 수 있다.  

#### include /etc/nginx/conf.d/service-env.inc;
- service-env.inc 파일 내부에 있는 변수 service_url을 default.conf 파일 내부에 삽입한다.  

#### upstream green or blue
- nginx의 upstream은 로드 밸런싱을 위한 서버 그룹(백엔드 서버들)을 정의하는 블록이다.  
- 주로 리버스 프록시 설정에서 사용되어, 하나의 도메인에 대해 여러 서버에 트래픽을 분산시킬 때 유용하다.  
- 변수 service_url의 값에 따라 요청 트래픽이 green 또는 blue의 환경의 프라이빗 IP로 전달된다.  

<br>

<img src="/assets/image/projects/ci-cd-zero-downtime-auth/zero-downtime-deployment-4.png" alt="empty" style="width: 50%; height: auto;">  

- 8080 혹은 8081 포트 없이 Nginx에 의해 서버 연동이 완료된 것을 알 수 있다.  

## 애플리케이션 프로필과 도커 파일
```
spring:
  profiles:
    active: local
    group:
      local: local, common, dev
      green: green, common, prod
      blue: blue, common, prod

server:
  env: local

---

spring:
  config:
    activate:
      on-profile: local

server:
  name: local_server
  host: localhost
  port: 8080

---

spring:
  config:
    activate:
      on-profile: green

server:
  name: green_server
  host: ${SERVER_HOST}
  port: 8080

---

spring:
  config:
    activate:
      on-profile: blue

server:
  name: blue_server
  host: ${SERVER_HOST}
  port: 8081

---
```

여러 서버에 애플리케이션을 분산 배포할 때는 Nginx 설정뿐만 아니라, 각 애플리케이션 서버에서도 자신이 속한 서버 정보를 명시해야 한다. 동일한 프로그램을 여러 환경에 배포하는 경우, 이 정보는 컴파일 시점에 하드코딩하기보다는 실행 시점에 동적으로 주입하는 방식이 바람직하다.  

위 코드는 Spring의 application.yml 파일에 정의된 서버 설정이다. 자세히 보면 on-profile 항목이 local, green, blue로 나뉘어 있으며, 각 프로파일에 따라 서버 설정 값이 달라진다.  

그렇다면 실행 환경에 따라 적절한 서버 정보를 어떻게 주입할 수 있을까? 그 해답은 도커파일(Dockerfile)에 있다. 도커는 실행 시점에 환경변수를 주입할 수 있어, 서버 정보나 프로파일을 컨테이너 실행 환경에 맞게 유연하게 설정할 수 있도록 해준다.  

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=build/libs/*.jar
ARG PROFILES
ARG ENV
COPY ${JAR_FILE} app.jar하다
ENTRYPOINT ["java", "-Dspring.profiles.active=${PROFILES}", "-Dserver.env=${ENV}", "-jar", "app.jar"]
```

위 코드는 도커파일(Dockerfile)의 내용이다. 코드 하단을 보면 ENTRYPOINT 명령어를 통해 PROFILES과 ENV 값이 주입되는 것을 확인할 수 있다.  

이 PROFILES과 ENV는 YML 설정 파일의 on-profile 및 env 항목과 연관되어 있으며, 결국 새로 배포되는 서버는 도커 빌드 과정에서 이러한 값들에 따라 사전에 정의된 환경으로 구성된다.  

## Profile API
```
@RestController
public class HealthCheckController {

    @Value("${server.name}")
    private String serverName;
    @Value("${server.host}")
    private String serverHost;
    @Value("${server.port}")
    private String serverPort;
    @Value("${server.env}")
    private String env;

    @GetMapping("/hc")
    public ResponseEntity<?> healthCheck() {
        Map<String, String> responseData = new LinkedHashMap<>();
        responseData.put("serverName", serverName);
        responseData.put("serverHost", serverHost);
        responseData.put("serverPort", serverPort);
        responseData.put("env", env);

        return ResponseEntity.ok(responseData);
    }

    @GetMapping("/env")
    public ResponseEntity<?> getEnv() {
        return ResponseEntity.ok(env);
    }
}
```

현재 실행 중인 Spring Boot의 active profiles의 관련 정보를 가져오고, 요청받은 profile이 포함되어 있는지를 확인하여 반환하는 RestController 예제이다. 이를 통해 브라우저에서 현재 어떤 profile이 활성화되어 있는지 확인할 수 있다.  

## 무중단 배포 스크립트
### Build - Build and push Docker images
```
    build:
      - name: Build and push Docker images
        uses: docker/build-push-action@v6.15.0
        with:
          context: .
          push: true
          tags: ---------------------
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

EC2에 배포하기 위해 도커 파일을 빌드 및 허브에 PUSH 한다.  

### Deploy - Set Target IP #1
```
  deploy:
    steps:
      - name: Set Target IP
        # 1. curl은 서버 http 요청을 보내는 명령어
        # 2. -o /dev/null
        # -o는 출력 파일 지정, /dev/null은 "아무것도 저장하지 않음" → 즉, 응답 본문을 버림
        # 3. -w "%{http_code}"
        # -w는 --write-out의 약자
        # 요청 결과의 특정 값만 출력할 수 있음
        # %{http_code}는 HTTP 응답 코드만 출력 (예: 200, 404, 500)
        # -> GitHub Actions에서 비밀 주소로 된 서버에 요청을 보내고, 
        # 서버가 살아있는지 확인하기 위해 HTTP 응답 코드를 STATUS 변수에 저장하는 코드
        run: | 
          STATUS=$(curl -o /dev/null -w "%{http_code}" "http://${{ secrets.SERVER_HOST }}/env")
          echo $STATUS
          if [ $STATUS = 200 ]; then
            CURRENT_UPSTREAM=$(curl -s "http://${{ secrets.SERVER_HOST }}/env")
          else
            CURRENT_UPSTREAM=blue
          fi
          
          echo CURRENT_UPSTREAM=$CURRENT_UPSTREAM >> $GITHUB_ENV
          if [ $CURRENT_UPSTREAM = green ]; then
            echo "CURRENT_PORT=8080 >> $GITHUB_ENV
            echo "STOPPED_PORT=8081 >> $GITHUB_ENV
            echo "TARGET_UPSTREAM=blue" >> $GITHUB_ENV
          else
            echo "CURRENT_PORT=8081 >> $GITHUB_ENV
            echo "STOPPED_PORT=8080 >> $GITHUB_ENV
            echo "TARGET_UPSTREAM=green" >> $GITHUB_ENV
          fi
```

1. 두 서버 모두 가동되고 있지 않다면 HTTP 상태 코드를 통해 하나의 서버를 가동하기.  
2. 하나의 서버 (ex green)가 가동 중이라면 배포될 서버로 이름과 PORT 번호를 변경한다.  

### Deploy - Docker compose up #2
```
  deploy:
    steps:
      - name: Docker compose up
        uses: appleboy/ssh-action@v1.2.1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          script: |
            mkdir -p /home/ubuntu/ci-cd-authentication
            cd /home/ubuntu/ci-cd-authentication
            echo '${{ secrets.APPLICATION_ENV_FILE }}' > .env

            if [ '${{ env.TARGET_UPSTREAM }}' = green ]; then 
              echo '${{ vars.DOCKER_COMPOSE_GREEN_YML }}' > docker-compose.yml
            else
              echo '${{ vars.DOCKER_COMPOSE_BLUE_YML }}' > docker-compose.yml
            fi
            
            docker compose up -d --pull always
            
            rm -rf docker-compose.yml
            rm -rf .env
```

배포될 서버의 도커 이미지를 가져와 실행시킨다.  

### Deploy - Check the deployed service URL #3
```
  deploy:
    steps:
      - name: Check the deployed service URL
        uses: jtalk/url-health-check-action@v4
        with:
          url: http://${{ secrets.SERVER_HOST }}:${{ env.STOPPED_PORT }}/env
          max-attempts: 7 # Optional, defaults to 1
          retry-delay: 10s # Optional, only applicable to max-attempts > 1
```

새로 가동된 서버가 정상적으로 동작하는지 확인하기 위해, URL을 통한 이른바 Health Check 검사를 수행한다. 만약 지정된 횟수만큼 서버가 응답하지 않으면, 배포를 중단하고 기존 서버로 롤백한다.  

### Deploy - Change nginx upstream #4
```
  deploy:
    steps:
      - name: Change nginx upstream
        uses: appleboy/ssh-action@v1.2.1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          # 1. 컨테이너 안에서 bash 셸을 실행하고 -c로 전달된 명령을 실행
          # 2. 지정된 경로 service_url에 TARGET_UPSTREAM 값 대입
          # 3. 설정을 수정한 후, nginx를 다시 로드해 변경 사항을 적용
          script: |
            docker exec -i my-nginx bash -c 'echo "set \$service_url ${{ env.TARGET_UPSTREAM }};" > /etc/nginx/conf.d/service-env.inc && nginx -s reload'
```

새로 배포된 Blue 서버로 트래픽을 전환하기 위해 Nginx의 /etc/nginx/conf.d/service-env.inc 경로의 $service_url 값을 Blue로 변경한다.  

### Deploy - Stop current Server #5
```
  deploy:
    steps:
      - name: Stop current Server
        uses: appleboy/ssh-action@v1.2.1
        with:
          host: ${{ secrets.SSH_HOST }}
          username: ${{ secrets.SSH_USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: 22
          script: |
            docker stop ${{ env.CURRENT_UPSTREAM }}
            docker rm ${{ env.CURRENT_UPSTREAM }}
            docker image prune -a -f
```

배포된 서버가 정상적으로 가동되고 있고, Nginx 설정을 통해 트래픽을 새 서버로 전환했기 때문에 기존 서버는 종료한다.  

## 참고
- [[Spring 배포 #4] EC2 인스턴스 Nginx로 블루-그린 무중단 배포하기](https://loosie.tistory.com/780#%EB%AC%B4%EC%A4%91%EB%8B%A8_%EB%B0%B0%ED%8F%AC%EB%9E%80?)
- [[서비스 배포 전략 구상하기] 무중단 배포 3가지 방식 (Rolling, Blue-Green, Canary)](https://loosie.tistory.com/781)
- [도커를 이용한 웹서비스 무중단 배포하기](https://subicura.com/2016/06/07/zero-downtime-docker-deployment.html)
- [Published 2024. 4. 16. 17:17 L4 스위치의 A to Z (Load Balancing, Traffic Flow, SSL Offload 등)](https://kangmanjoo.tistory.com/161)