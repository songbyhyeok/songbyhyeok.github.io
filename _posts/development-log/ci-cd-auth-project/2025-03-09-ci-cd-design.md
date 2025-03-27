---
title: 2 - CI/CD 설계
categories: ci-cd-auth-project
---

# CI/CD Workflow

<img src="https://github.com/user-attachments/assets/110d7e0e-6a24-4d76-a867-fd6b7bb0bf0c" alt="empty" loading="lazy" style="width: 800px; height: auto;">  

위 다이어그램은 Spring 환경에서 적용된 GitHub Actions CI/CD Workflow 이다. 전체 프로세스는 다음과 같다.  

1. CI/CD 구축을 위해 하나의 workflow를 생성하고 설계하였다. 이제 CI/CD workflow 안에서 특정 이벤트나 작업을 설정하여 자동화할 수 있게 있다. 
2. 내가 의도한 대로 trigger가 작동하도록 이벤트에 Push와 Pull Request를 설정하였다. 이를 통해 Git 관리나 레포지토리 코딩 단계에서 CI/CD가 자동으로 작동하게 된다. 
3. 운영과 관리를 효율적으로 하기 위해 CI와 CD로 작업(JOB)을 분리하였다. 이로써 CI가 먼저 실행된 후, 그 다음에 CD가 실행되도록 구성하였다.
4. CI 작업에서는 레포지토리의 코드에 JDK 17과 Gradle을 설치하고, 빌드 작업을 수행하였다. 이후 단계에서는 효율적인 빌드 관리와 애플리케이션 개발이 가능해진다.
5. 두 번째 CD 작업에서는 Docker를 사용해 CI에서 가져온 빌드된 파일을 가져와서 DockerHub에 업로드하였다. 이후, EC2 인스턴스에서 DockerHub에 저장된 이미지를 가져와 Docker 컨테이너에서 실행하기 위해 SSH 터널을 구축하여 EC2에 접속하였다.  

<br>

## .env 파일 이식하기
### PlaceholderResolutionException Issue
1. GitHub Actions CI Build 작업 중에 PlaceholderResolutionException Error가 발생하였다.
2. 이 오류는 주로 application의 환경 파일에서 설정된 프로퍼티 값이 제대로 연동되지 않을 때 발생하게 된다.
3. Local 상에서는 인텔리제이 IDE에 외부 .env 환경변수 파일을 주입시켜 환경파일의 설정 값들과 연동시켰는데, githubActions 상에서는 환경변수 파일을 주입시켰음에도 인식되지 않아 build 과정에서 문제가 발생하게 되었다.
4. 크게 세 가지 방법 Dotenv 라이브러리, workflow에 .env 파일 주입을 위한 코드 작성, application.yml 파일에 .env 인식 설정이 있다. 이 3가지 중에 application.yml에 .env 인식 설정 방법을 사용하게 되었다.
* Dotenv
  * 라이브러리 설치, 설정 그리고 .env 파일을 시스템 속성에 코드를 통해 주입을 해야 하기 때문에 번거롭고 생산 비용을 요구한다.
* workflow에 .env 파일을 코드를 통해 주입
  * GitHub Actions에서 사용되는 기본 스크립트 언어는 Bash이고, 나에게는 낯설다. 그리고 workflow에 코드를 통해 주입 과정에서 시행착오가 필요하므로 많은 시간적 비용을 요구한다.
* application.yml에 .env 인식
```
spring:
  config:
    import: optional:file:.env[.properties]
```
  * 위 코드를 application 환경파일에 기입하면 github actions는 .env 파일을 인식하게 된다. 두 방법과 비교해서 간단하고 효율적이기 때문에 이 방법을 채택하게 되었다.

<br>

## GitHub Actions 캐시 최적화
### workflow 결과를 도출하기 위해 소모되는 시간적 비용은 크다.
* workflow 결과를 도출하기 위해 action이 실행되고 이를 기다리는 시간은 30s ~ 2m 정도의 비용을 요구한다.

### github actions에서 제공하는 cache action을 사용하여 캐시 최적화를 시도
![Image](https://github.com/user-attachments/assets/ec13b088-59dd-451a-8fe5-73601eedf5a0)  

```
      # 작업 중에 생성된 파일이나 디렉토리를 캐시하여, 이후에 해당 파일들을 재사용함으로써 빌드 시간을 단축
      - uses: actions/cache@v4
        with:
          path: ~/.gradle  # Gradle 캐시 디렉토리
          key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle/wrapper/gradle-wrapper.properties') }} # 캐시 키
          restore-keys: | # 캐시 복원 시 사용할 백업 키
            ${{ runner.os }}-gradle- 

      # Gradle을 자동으로 설치하고 설정하는 데 사용
      - name: Setup Gradle
        # 이전의 cache가 없다면 의존성을 설치합니다.
        if: steps.cache.outputs.cache-hit != 'true'
        uses: gradle/actions/setup-gradle@v4

      # Gradle을 사용하여 빌드 실행
      - name: Build with Gradle
        run: ./gradlew build
```

1. 시간적 비용을 줄이기 위해 **`actions/cache@v4`** 을 사용해서 gradle setup과 build 시간 단축을 시도하였다.
2. 제공된 cache@v4 템플릿의 gradle 경로와 cache가 저장될 key 값을 지정하기 위해 실제 spring boot를 build 했을 때 생성되는 빌드 값과 관련 파일들의 경로가 어디로 생성되는지 분석하였다.
3. gradle setup과 build 이전에 캐시 값이 먼저 부를 수 있게 순서를 맞추고 캐시 값을 성공적으로 불러들였다면, setup이 되지 않게 조건문을 넣어 캐싱 처리를 하였다.

<br>

**참고한 자료**  
- [GitHub Actions 캐시 조금 더 다뤄보기 (feat. restore-keys)](https://pozafly.github.io/dev-ops/cache-and-restore-keys-in-github-actions/)
- [뱅크샐러드 Web chapter에서 GitHub Action 기반의 CI 속도를 개선한 방법](https://blog.banksalad.com/tech/github-action-npm-cache/)

<br>

## GitHub Actions에서 EC2 인스턴스에 SSH 연결을 허용하기
### dial tcp i/o timeout Issue
1. EC2 인스턴스에 접근하기 위해 SSH 연결 시도 중 dial tcp i/o timeout 에러가 발생하였다.
2. 이 오류는 GitHub Actions에서 특정 네트워크 요청이 시간 내에 완료되지 않아서 발생한 문제인데, 보통 외부에서 접근하는 IP를 허용하지 않는 서버 방화벽에 의해서 차단된 것이다.
3. EC2를 관리하는 Subnet의 보안그룹의 인바운드에 ssh 허용 Ip 주소를 깃허브를 포함시킬 수 있게 전체로 바꾸어 이를 해결하였다.

<br>

## Docker 학습 
### Dockerfile 생성 및 빌드
```
chmod +x gradlew
./gradlew build && java -jar build/libs/*.jar

docker build --build-arg JAR_FILE=build/libs/\*.jar -t songbyhyeok/ci-cd-auth .
docker run -d -p 8080:8080 --env-file .env songbyhyeok/ci-cd-auth .

chmod +x gradlew
* gradlew 파일에 실행 권한을 추가하는 명령어
* ./gradlew build 권한을 허용한다.

./gradlew build && java -jar build/libs/*.jar
* 현재 디렉토리에서 Gradle Wrapper(gradlew)를 사용하여 build 작업을 실행하는 명령어
* 프로젝트의 소스 코드를 빌드하고, build/libs/ 디렉토리에 JAR 파일을 생성
* java -jar build/libs/*.jar는 build/libs/ 디렉토리에 생성된 JAR 파일을 실행하는 명령어

docker build --build-arg JAR_FILE=build/libs/\*.jar -t songbyhyeok/ci-cd-auth .
* JAR_FILE 인수를 build/libs/*.jar로 설정하고, Dockerfile을 사용해 Docker 이미지를 빌드

docker run -d -p 8080:8080 --env-file .env songbyhyeok/ci-cd-auth .
* 빌드한 Docker 이미지를 기반으로 컨테이너를 실행하는 명령어
* -d: Detached mode로 실행하여 컨테이너가 백그라운드에서 실행
* -p 8080:8080: 호스트의 8080 포트와 컨테이너의 8080 포트를 매핑하여 연결
* --env-file .env: .env 파일에서 환경 변수를 로드하여 컨테이너에 전달
* 프로젝트명: 실행할 Docker 이미지 이름 지정
* 태그: 지정된 태그 명으로, 로컬에 이미지가 없으면 자동으로 Docker Hub에서 이미지를 다운로드
```

```
FROM openjdk:17-jdk-alpine
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]

FROM 
* 기본 이미지를 설정
ARG 
* ARG는 빌드 시에 사용할 수 있는 빌드 인수를 정의 
* JAR_FILE이라는 변수를 선언하고, 기본값으로 build/libs/*.jar를 지정
* JAR_FILE은 Docker 이미지 빌드 시에 사용
COPY 
* COPY 명령어는 호스트 시스템에서 컨테이너로 파일을 복사
* JAR_FILE에 해당하는 파일(즉, target/*.jar)을 app.jar라는 이름으로 컨테이너의 루트 디렉토리에 복사
ENTRYPOINT
* 컨테이너가 시작될 때 실행될 명령어를 지정
* java -jar /app.jar 명령을 실행하여 컨테이너가 실행될 때 JAR 파일(app.jar)을 Java로 실행
```

1. 배포할 애플리케이션을 효율적으로 운용하기 위해 EC2 Ubuntu 환경 위에 컨테이너 Docker를 사용하기로 결정하였다. 도커를 통해 애플리케이션을 컨테이너화하여 배포 및 관리의 효율성을 높일 수 있기 때문이다.  

2. Docker에서 애플리케이션을 실행하기 위해서는 이미지를 기반으로 컨테이너를 생성해야 하는데, 이때 이미지를 정의하는 파일이 바로 Dockerfile이다. Dockerfile을 생성하기 위해 Spring 공식 문서에서 제공하는 샘플 Dockerfile을 참고하였다. 이를 통해 기본적인 Dockerfile 작성 규칙과 설정 방법을 이해하고, 바로 적용하였다.
    
3. 생성된 도커파일을 Build할 차례이다. 도커 공식문서 메뉴얼을 참고하여 명령어를 작성하였다.

<br>

**참고 자료**
- [Writing a Dockerfile](https://docs.docker.com/get-started/docker-concepts/building-images/writing-a-dockerfile/)
- [Build, tag, and publish an image](https://docs.docker.com/get-started/docker-concepts/building-images/build-tag-and-publish-an-image/)
- [Spring Boot with Docker](https://spring.io/guides/gs/spring-boot-docker)

<br>

## EC2 우분투 환경에서 Docker 컨테이너 실행하기
### Docker Build
```
    steps:
      # 이전 단계에서 업로드한 빌드 아티팩트를 다운로드하여, 이후 배포나 다른 작업에 사용할 수 있게 준비하는 역할
      - name: Download a Build Artifact
        uses: actions/download-artifact@v4.1.9
        with:
          name: jar-file
          path: build/libs/

      - name: Login to Docker Hub
        uses: docker/login-action@v3.3.0
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_TOKEN }}

      - name: Docker Setup Buildx
        uses: docker/setup-buildx-action@v3.10.0

      - name: Create Dockerfile
        env:
          docker_file: ${{ secrets.DOCKER_FILE }}
        run: |
            echo "${docker_file}" > ./Dockerfile

      - name: Build and push Docker images
        uses: docker/build-push-action@v6.15.0
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_HUB_TAG }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          
        context: .
        * 도커 이미지 빌드할 때 컨텍스트 지정
        * .은 현재 디렉터리
        push: true
        * 이 값이 true로 설정되면, 빌드가 완료된 후 Docker 이미지를 자동으로 푸시
        tags: ${{ secrets.DOCKER_HUB_TAG }}
        * Docker 이미지에 태그를 지정
        cache-from: type=gha
        * 빌드 캐시를 GitHub Actions 캐시(gha)에서 가져오도록 지정
        * type=gha는 GitHub Actions의 기본 제공 캐시를 사용하여, 이전에 빌드한 이미지를 캐시로 활용
        cache-to: type=gha,mode=max
        * 빌드 후 생성된 캐시를 GitHub Actions 캐시로 저장
        * type=gha는 캐시를 GitHub Actions 캐시 시스템에 저장
        * mode=max는 가능한 한 많은 캐시를 저장하겠다는 의미
```

1. CI 단계에서 Build한 jar-file을 Docker Hub에 업로드하기 위해 빌드 아티팩트를 사용해서 가져왔다.
2. 도커 허브에 로그인하기 앞서 비밀번호는 보안을 고려하여 토큰을 사용해서 비밀번호를 대체하였다.
3. Docker Setup Buildx는 고급 빌드 기능을 제공하는 도구이다. 이 도구는 다양한 고급 기능을 제공하는데, 이 중에 빌드 캐시를 활용하여 빌드 성능을 최적화 할 수 있어서 이 도구를 사용하였다.
4. Dockerfile을 사용하기 위해 GitHub Secrets에 저장된 비밀 값에서 가져와 변수에 이식시켰다.
5. 도커 이미지 빌드 및 도커 허브에 업로드하기 위해 관련 action을 사용하였고, 캐시를 사용해서 빌드 속도를 최적화하였다.

### EC2에서 Docker 컨테이너 실행
```
       - name: SSH Remote Commands
        uses: appleboy/ssh-action@v1.2.1
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.SSH_PORT }}
          script: |
            mkdir -p /home/ubuntu/ci-cd-authentication
            cd /home/ubuntu/ci-cd-authentication
            
            echo '${{ secrets.APPLICATION_ENV_FILE }}' > .env

						sudo docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKER_HUB_TOKEN }}
            sudo docker rm -f ${{ secrets.DOCKER_CONTAINER_NAME }}
            sudo docker images -q "${{ secrets.DOCKER_HUB_TAG }}" && sudo docker rmi -f "${{ secrets.DOCKER_HUB_TAG }}"
            sudo docker run -d -p 8080:8080 --env-file .env --name ${{ secrets.DOCKER_CONTAINER_NAME }} ${{ secrets.DOCKER_HUB_TAG }}
            
            sudo docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKER_HUB_TOKEN }}
            * -u: Docker Hub에서 사용할 사용자 이름을 지정
            * -p: Docker Hub에서 사용할 패스워드 또는 토큰을 지정
            sudo docker rm -f ${{ secrets.DOCKER_CONTAINER_NAME }}
            * Docker 컨테이너 삭제 명령어
            * -f: 강제로 실행 중인 컨테이너를 종료하고 삭제
            sudo docker images -q "${{ secrets.DOCKER_HUB_TAG }}" && sudo docker rmi -f "${{ secrets.DOCKER_HUB_TAG }}"
            * 지정된 태그를 가진 Docker 이미지의 ID를 조회
            * rmi는 Docker 이미지를 삭제하는 명령어
            * -f: 강제로 이미지를 삭제
            sudo docker run -d -p 8080:8080 --env-file .env --name ${{ secrets.DOCKER_CONTAINER_NAME }} ${{ secrets.DOCKER_HUB_TAG }}
            * docker run: 새로운 Docker 컨테이너를 실행하는 명령어
            * -d: 백그라운드에서 컨테이너를 실행하도록 지정
            * -p 8080:8080: 호스트의 8080 포트를 컨테이너의 8080 포트와 연결
            * --env-file .env: .env 파일에 정의된 환경 변수를 컨테이너에 전달
            * --name ${{ secrets.DOCKER_CONTAINER_NAME }}: 컨테이너의 이름을 설정
            * 마지막: Docker Hub에서 가져올 이미지 태그, 이 태그를 가진 이미지를 사용하여 컨테이너를 실행
```

1. 보안이 보장된 환경인 SSH를 통해 EC2에 접근하기 위해서 EC2 정보와 SSH 비밀키, PORT 등을 설정하였다.
2. 작업할 프로젝트 폴더에 접근하기 위해 생성 및 이동 명령어를 넣어주었다.
3. APP에 사용될 환경변수 파일을 도커 컨테이너에 전달하기 위해 secrets에서 가져와 EC2에 이식하였다.
4. 매 CD 작업마다 쌓이는 이미지와 컨테이너를 종료 및 삭제하고 새로 실행하게끔 만들기 위해 -q와 -f 옵션을 삭제 명령어에 추가해서 해결하였다.
5. 도커 허브에서 빌드 파일을 가져와서, 사용되고 있는 포트 번호와 아까 이식한 환경변수 파일을 도커 컨테이너에 포함시켜 실행하게 만들었다.