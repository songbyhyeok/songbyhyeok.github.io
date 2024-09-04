---
title: Github Actions CI/CD Review
categories: Review
tag: [ISSUE, Review, Github Actions, CI/CD, AWS, EC2, RDS, Docker]
---

![image](https://github.com/user-attachments/assets/817594f2-afe3-4ed9-a49a-1f8bec26e7d4)

Github Actions를 사용해서 CI/CD System을 구축하였다. 원래는 기획 단계의 개발 마무리 단계에서 진행하려고 했지만 초기에 자동화 시스템을 구축해 놓는 것이 맞겠다는 판단이 들어 계획을 틀었다. 진행하면서 용어가 낯설어 공부하는데 진을 뺐고, 필요 기능을 가져다 사용하려고 해도 많은 자료가 여러 버전의 형식으로 제공하고 있어서... 낯선 Shell 문법을 오려다 붙인다는 것이 얼마나 두렵던지 "이게 맞아? 이렇게 하면 되는 건가? 아닌 거 같은데?" 이런 생각들로 실행에 주춤하게 되었고 내 혼을 쏙 뺐다. 하지만 심리학에서 그랬듯 "단순노출효과"로 인하여 자주 보는 것만으로 shell하고 친구가 될 수 있었다.

## EC2 설정
* **OS**  
Amazon Linux를 채택했었지만, 설정 과정에서 호환 이슈로 aws cli를 통한 설치를 해야 하는 어려움이 있어 ubuntu로 바꾸었다.  
* **Keypair**  
RSA와 Ed25519중 Ed25519를 채택했다. 둘 다 공개키 알고리즘이지만 aws에서 window에서는 ed25519를 지원하지 않아 linux 전용으로 생각이 들었고, 그리고 비교적 rsa보다 안전성과 속도 면에서 좀 더 빠르다.  
* **보안**  
인바운드에 http, https, ssh, spring, mysql의 port만 허용하였다.  
* **탄력적 IP**  
프리티어 등급이라 하나의 인스턴스에 공용 ip인 탄력적 ip를 연동시켰다.

## RDS 설정
* **DBMS**  
MySQL을 채택하였다. RDBMS에서 대중적이고 가볍고 사용해봤기 때문이다.   
* **IP**  
공개 ip를 생성시켜서 외부에서 접근이 가능하게 했었지만, ipv4 고갈 이슈로 과금의 원인이 되어 private 상태로 두어 port forwarding 방식으로 ec2에서 접근할 수 있게 만들었다.

## GitHub Actions Workflow
    name: Java CI/CD with Gradle and Docker

    on:
    push:
        branches: [ "feat/#27/Github-Actions-CI/CD" ]
    pull_request:
        branches: [ "develop" ]

    jobs:
    ci:
        runs-on: ubuntu-latest
        permissions:
        contents: read
        steps:
        # workflow가 저장소에 접근 허용
        - name: Checkout sources
            uses: actions/checkout@v4

        # JDK 버전 설치 및 이곳에 배포
        - name: Set up JDK 17
            uses: actions/setup-java@v4
            with:
            distribution: 'temurin'
            java-version: '17'

        # Build 과정에 사용될 환경 파일들을 생성
        - name: Create Application and Envfile
            env:
            env_file: ${{ secrets.LOCAL_ENV_FILE }}
            application_yaml: ${{ secrets.APPLICATION_YAML }}        
            run: |
            echo "${env_file}" > ./.env
            
            mkdir -p ./src/main/resources
            cd ./src/main/resources
            echo "${application_yaml}" > application.yaml

        # ubuntu 환경에서 Gradle 접속 허가
        - name: Grant execute permission for Gradle
            run: chmod +x ./gradlew

        # Gradle 설정 및 캐시 최적화
        - name: Setup Gradle
            uses: gradle/actions/setup-gradle@v3
            with:
            gradle-version: wrapper

        - name: Cache Gradle packages
            uses: actions/cache@v4
            with:
            path: |
                ~/.gradle/caches
                ~/.gradle/wrapper
            key: ${{ runner.os }}-gradle-${{ hashFiles('**/*.gradle*', '**/gradle-wrapper.properties') }}
            restore-keys: |
                ${{ runner.os }}-gradle-

        # local to private RDS 목적의 로컬 포트 포워딩
        - name: Setup SSH key
            env: 
            ssh_hostname: ${{ secrets.SSH_HOSTNAME }}
            ssh_key_file: ${{ secrets.SSH_KEY_FILE }}
            run: |
            mkdir -p ~/.ssh
            echo "${ssh_key_file}" > ~/.ssh/id_ed
            chmod 600 ~/.ssh/id_ed
            ssh-keyscan -H ${ssh_hostname} >> ~/.ssh/known_hosts

        - name: Start SSH tunnel
            env:
            local_db_port: ${{ secrets.LOCAL_DB_PORT }}
            rds_endpoint: ${{ secrets.RDS_ENDPOINT }}
            rds_port: ${{ secrets.RDS_PORT }}
            ssh_username: ${{ secrets.SSH_USERNAME }}
            ssh_hostname: ${{ secrets.SSH_HOSTNAME }}
            run: |
            nohup ssh -i ~/.ssh/id_ed -fN -L ${local_db_port}:${rds_endpoint}:${rds_port} ${ssh_username}@${ssh_hostname} &
            echo $! > ssh-tunnel.pid
            echo "SSH tunnel started with PID $(cat ssh-tunnel.pid)"

        # CI/CD 환경에서 격리됐기 때문에 daemon의 캐시 기능은 불필요하다고 판단, 오버헤드를 줄이기 위해 설정
        - name: Build with Gradle
            run: ./gradlew build --no-daemon

        # 쓰레드 cd가 ci의 jdk를 사용해야 하기 때문에 사용한다.
        - name: Upload artifact
            uses: actions/upload-artifact@v4
            with:
            name: jar-file
            path: build/libs/*.jar

        # jvm 빌드가 끝났기 때문에 터널을 닫아준다.
        - name: Stop SSH tunnel
            env:
            local_db_port: ${{ secrets.LOCAL_DB_PORT }}
            rds_endpoint: ${{ secrets.RDS_ENDPOINT }}
            rds_port: ${{ secrets.RDS_PORT }}
            run: |
            echo "Stopping SSH tunnel"
            if [ -f ssh-tunnel.pid ]; then
                PID=$(cat ssh-tunnel.pid)
                echo "Killing SSH tunnel with PID $PID"
                kill $PID || true
                sleep 2
                if pgrep -f "ssh -i ~/.ssh/id_ed -fN -L ${local_db_port}:${rds_endpoint}:${rds_port}" > /dev/null; then
                echo "SSH tunnel still running. Forcing termination."
                kill -9 $PID || true
                else
                echo "SSH tunnel stopped successfully"
                fi
                rm ssh-tunnel.pid
            else
                echo "No PID file found. SSH tunnel might not have been started."
            fi
    cd:
        runs-on: ubuntu-latest
        permissions:
        contents: read
        needs: ci
        steps:
        # ci에서 build된 jar 파일을 가져온다.
        - name: Download artifact
            uses: actions/download-artifact@v4
            with:
            name: jar-file
            path: build/libs/
        
        # Docker Build 설정과 Hub 연동
        - name: Set up Docker Buildx
            uses: docker/setup-buildx-action@v3

        - name: Login to Docker Hub
            uses: docker/login-action@v3
            with:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}

        # 빌드에 필요한 도커파일과 제외 설정을 가져온다.
        - name: Create Dockerfile and .dockerignore
            env:
            docker_file: ${{ secrets.DOCKER_FILE }}
            docker_ignore: ${{ secrets.DOCKER_IGNORE }}
            run: |
            echo "${docker_file}" > ./Dockerfile
            echo "${docker_ignore}" > ./.dockerignore
            ls -R

        # 빌드하여 이미지 생성 및 Hub에 업로드
        - name: Build and push
            uses: docker/build-push-action@v6
            with:
            context: .
            file: Dockerfile
            push: true
            tags: ${{ secrets.DOCKERHUB_IMAGENAME }}:latest
            build-args: |
                JAR_FILE=build/libs/*.jar
            # - GitHub Actions cache 적용 -
            # 매 작업은 새 러너로 작동되기 때문에 종속성의 비용을 GitHub worfklow cache를 사용해 시간 단축
            # gha는 actions 전용 캐시 기능이다.
            # mode=<min or max> 내보낼 캐시 레이어
            # min -> 결과 이미지에 대한 레이어만 내보낸다.
            # max -> 모든 중간 단계의 모든 레이어를 내보낸다.
            cache-from: type=gha
            cache-to: type=gha,mode=max

        # ssh로 ec2에 접속해서 hub의 이미지를 가져와 컨테이너를 구동
        - name: Deploy to AWS EC2
            uses: appleboy/ssh-action@v1.0.3
            with:
            host: ${{ secrets.SSH_HOSTNAME }}
            username: ${{ secrets.SSH_USERNAME }}
            key: ${{ secrets.SSH_KEY_FILE }}
            port: ${{ secrets.SSH_PORT }}
            script: |
                docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_TOKEN }}

                mkdir -p /home/ubuntu/giwoot_jang
                cd /home/ubuntu/giwoot_jang
                
                docker stop $(docker ps -a -q)
                docker rm $(docker ps -a -q)
                docker rmi $(docker images -q)
                
                docker pull ${{ secrets.DOCKERHUB_IMAGENAME }}
                echo '${{ secrets.DOCKER_ENV_FILE }}' > .env
                echo '${{ secrets.DOCKER_COMPOSE_YAML }}' > compose.yaml           
                docker compose -f compose.yaml up -d
                
                rm -rf /home/ubuntu/giwoot_jang/

                # 현재 세션의 명령 기록을 지우기
                history -c
                # 기존 기록 파일 삭제하기
                rm ~/.bash_history
                # 현재 세션의 빈 기록을 새 파일에 저장하기 (파일이 새로 생성될 수 있음)
                history -w