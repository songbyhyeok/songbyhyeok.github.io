---
title: AWS RDS IP 과금 이슈 및 로컬 포트 포워딩 RDS연동과 Github Action Workflow Build
categories: Troubleshooting
tag: [ISSUE, AWS, RDS, MYSQL]
---

![image](https://github.com/user-attachments/assets/f85a6436-1c39-4c3b-989f-c1fd4fffcf22)

지난 3 ~ 7월까지 매달 5달러 가량을 AWS에 지불하고 있었다. 그 원인이 무엇인지는 비용이 그리 크지 않아 그냥 방치하고 있었지만, 쌓이면 그래도 무시못할 금액이기에 이를 해결하고자 나섰다.

## 원인 파악하기
![image](https://github.com/user-attachments/assets/28d6b763-cf9c-4798-9e34-721075870d5d)

단순 VPC 부분이 문제인 것은 알겠지만 구체적으로 어디서 무엇이 어떻게 작동되어 비용이 발생하는 것인지는 파악하기 힘들어 나와 같은 사례를 찾게 되었고 어떤 한 분이 고맙게도 똑같은 현상에 대해 해결 방법을 글로 써주셔서 해결할 수 있게 되었다. 대강 원인과 해결 방법은 Public RDS가 새로운 IPv4를 생성해서 비용이 발생했고, 이를 Private 상태로 변경시켜 IP를 사용하지 않으면 되는 것이었다.(IPv4 고갈로 인한... 조치)

## 로컬 포트 포워딩으로 RDS 연결하기.
기존 방식은 생성된 rds ip로 연동시키면 됐지만, 이제는 ssh 방식으로 ec2에 접근해서 우회하여 rds에 연동시켜야 한다.

**Intellij DataGrip**  
![image](https://github.com/user-attachments/assets/ea1d2c4f-c8ac-4cc0-b326-6eed5723553f)

숫자 순번대로 클릭해 넘어가서 접속할 SSH 설정을 입력하면 된다. 비대칭 키를 사용했었기 때문에 Host는 ec2 ip, Username은 os명, key 방식 및 file.pem을 지정시킨다.

![image](https://github.com/user-attachments/assets/743ea187-0a33-451d-bdaf-ef9989308b75)

그다음 General 탭으로 들어가서 RDS 정보를 입력하면 된다. Host는 엔드포인트, user 및 비번 그리고 Database는 권한부여된 스키마명이다.

**MySQL Workbench**
![image](https://github.com/user-attachments/assets/b681b8cd-498f-4cd4-8cf8-089ceb45a96d)

DataGrip에 입력했던 데이터명을 참고해서 매핑할 것

**application.yml**  
가장 중요한 부분이 아닐까 싶다.  

    spring:
        datasource:
            url: jdbc:mysql://localhost:3307/${MYSQL_DATABASE}
            username: ${MYSQL_ID}
            password: ${MYSQL_PASSWORD}
            driver-class-name: com.mysql.cj.jdbc.Driver

사실 localhost:3307 대신 RDS엔드포인트를 입력해도 연결을 할 수 있다. 접근을 할 수 없는데 왜?인지는 솔직히 잘모르겠다;; 무튼 local ip를 입력하고 서버 연결을 시도하면 연결이 되지 않는다. 왜냐하면 SSH 터널링을 생성시켜야 하기 때문이다. 그러기 위해서는 shell 명령어로 서버를 구축을 해야 한다.  

    프로젝트 root -> git bash ->
    ssh -i [.pem] -fN -L [localhost:3307]:[Rds-endpoint]:3306 [ssh name]@[ssh host]  

ssh 터널을 생성시키면 spring 환경에서 접속할 수 있게 된다. port를 3307로 설정한 이유는 기존 3306이 사용 중이기 때문이다.

**github action workflow**  
ci/cd 구축 중이라면 가장 난감한 문제가 아닐까 싶다. 왜냐하면 application.yml -> ssh 설정 및 터널 생성 그리고 빌드 테스트 후 생성된 ssh 서버 종료까지 전부 다 해야 하기 때문이다.

      - name: Create Application.yml
        shell: bash
        env: 
          MYSQL_DATABASE: ${{ secrets.MYSQL_DATABASE }}
          MYSQL_ID: ${{ secrets.MYSQL_ID }}
          MYSQL_PASSWORD: ${{ secrets.MYSQL_PASSWORD }}
          SECURITY_NAME: ${{ secrets.SECURITY_NAME }}
          SECURITY_PASSWORD: ${{ secrets.SECURITY_PASSWORD }}
        run: |
          mkdir -p ./src/main/resources
          cd ./src/main/resources
          echo "${{ secrets.APPLICATION_YML }}" > application.yml

      - name: Setup SSH key # ssh 접근에 필요한 key를 옮기기
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_ED25519_PEM }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed
          chmod 600 ~/.ssh/id_ed
          ssh-keyscan -H ${{ secrets.SSH_HOST }} >> ~/.ssh/known_hosts

      - name: Start SSH tunnel # ssh 터널 생성 시 생성된 서버 프로세스 id 파일로 저장
        run: |
          nohup ssh -i ~/.ssh/id_ed -fN -L 3307:${{ secrets.MYSQL_ENDPOINT }}:3306 ${{ secrets.SSH_USERNAME }}@${{ secrets.SSH_HOST }} &
          echo $! > ssh-tunnel.pid
          echo "SSH tunnel started with PID $(cat ssh-tunnel.pid)"

      - name: Stop SSH tunnel # 빌드 테스트 이후 서버 프로세스 id가 정상적으로 종료가 됐는지 확인하기 위해 조건문 추가
        if: always()
        run: |
          echo "Stopping SSH tunnel"
          if [ -f ssh-tunnel.pid ]; then
            PID=$(cat ssh-tunnel.pid)
            echo "Killing SSH tunnel with PID $PID"
            kill $PID || true
            sleep 2
            if pgrep -f "ssh -i ~/.ssh/id_ed -fN -L 3307:${{ secrets.MYSQL_ENDPOINT }}:3306" > /dev/null; then
              echo "SSH tunnel still running. Forcing termination."
              kill -9 $PID || true
            else
              echo "SSH tunnel stopped successfully"
            fi
            rm ssh-tunnel.pid
          else
            echo "No PID file found. SSH tunnel might not have been started."
          fi

## 참고
- [link1](https://velog.io/@dev_hyun/AWS-%ED%94%84%EB%A6%AC%ED%8B%B0%EC%96%B4%EC%9D%B8%EB%8D%B0-%EB%8F%88%EC%9D%B4%EB%82%98%EA%B0%84%EB%8B%A4-RDS-Public-IPv4)
- [link2](https://velog.io/@kjyeon1101/Spring-%EC%88%98%EC%88%99%EA%B4%80-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0Spring-Boot-IntelliJ-MySQL-ssh)