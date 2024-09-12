---
title: SSM port forwarding, ssh & 공개키없이 접속하기
categories: aws
---

기존 SSH를 통한 EC2 연결 방식은 공개키를 개인이 가지고 있어야 하는 부담감과 다른 로컬에서 접속하려고 할 경우 공개키를 공유로 인한 귀찮음과 보안 걱정을 해야만 했다. 하지만 AWS의 SSM 서비스를 사용한다면 이런 고민을 해결 할 수 있다.

**SSM이란?**  
"AWS Systems Manager"의 약자로 AWS 애플리케이션 및 리소스를 관리하는 시스템이다.

## aws cli, ssm plugin 설치
[aws cli](https://docs.aws.amazon.com/ko_kr/streams/latest/dev/kinesis-tutorial-cli-installation.html)

[aws session mgr plugin](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)

## aws ec2 정책, iam 권한부여
**ec2 역할에 정책 부여**  

    <!-- AWS Systems Manager 서비스 핵심 기능을 활성화하기 위한 Amazon EC2 역할 정책입니다. -->
    <!-- 사용자, 그룹 및 역할에 AmazonSSMManagedInstanceCore를 연결할 수 있습니다. -->
    AmazonSSMManagedInstanceCore 

**iam 권한 부여**  

    <!-- aws cli를 통해 ssm에 접속하여 ec2에 접근할 수 있다. -->
    <!-- aws region = 지역, 계정 id = aws 프로필 계정 id, 인스턴스 id = ec2 인스턴스 id -->
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:StartSession"
                ],
                "Resource": [
                    "arn:aws:ec2:{지역}:{계정id}:instance/{인스턴스id}",
                    "arn:aws:ssm:{지역}:{계정id}:document/SSM-SessionManagerRunShell",
                    "arn:aws:ssm:{지역}::document/AWS-StartPortForwardingSessionToRemoteHost"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "ssm:TerminateSession",
                    "ssm:ResumeSession"
                ],
                "Resource": [
                    "arn:aws:ssm:*:*:session/${aws:userid}-*"
                ]
            }
        ]
    }

## AWS CLI 자격 증명
    1. aws -> iam -> 액세스 키 발급 및 csv 저장(노출x)
    2. aws configure 
        AWS Access Key ID [None]: 
        AWS Secret Access Key [None]: 
        Default region name [None]: 
        Default output format [None]:
    3. cat ~/.aws/credentials (액세스 키 파일에 기입됐는지 확인)
    4. aws sts get-caller-identity (aws 자격 증명 됐는지 확인)
       1. powerShell 일 경우 관리자 권한으로 실행

## SSM port forwarding 
**Shell**

    set -a
    source .env
    set +a

    EC2_INSTANCE_ID=${EC2_INSTANCE_ID}
    LOCAL_PORT=${LOCAL_PORT}
    RDS_PORT=${RDS_PORT}
    RDS_ENDPOINT=${RDS_ENDPOINT}

    aws ssm start-session \
        --target "${EC2_INSTANCE_ID}" \
        --document-name AWS-StartPortForwardingSessionToRemoteHost \
        --parameters host="${RDS_ENDPOINT}",portNumber="${RDS_PORT}",localPortNumber="${LOCAL_PORT}"

**PowerShell**

    # .env 파일에서 환경 변수 로드
    Get-Content .env | ForEach-Object {
        if ($_ -match '^(.+)=(.+)$') {
            Set-Item -Path "Env:$($Matches[1])" -Value $Matches[2]
        }
    }

    aws ssm start-session `
        --target "$env:EC2_INSTANCE_ID" `
        --document-name AWS-StartPortForwardingSessionToRemoteHost `
        --parameters host="$env:RDS_ENDPOINT",portNumber="$env:RDS_PORT",localPortNumber="$env:DB_PORT"

## SpringBoot Local 자동 연동 설정
shell or powerShell 둘 중 하나의 스크립트를 사용한다. 인텔리제이 터미널 default = powerShell이므로 powerShell을 추천한다.

**shell**
위 shell 스크립트를 .sh 확장자로 저장 및 Run/Debug Configurations에서 스크립트 등록

**powerShell** 
![image](https://github.com/user-attachments/assets/04fb8dda-038b-4a04-92d7-bd44ddc05194)

인텔리제이에서 powerShell 플러그인 설치 및 위 powerShell 스크립트를 .ps1 확장자로 저장 후 Run/Debug Configurations에서 스크립트 등록

**multirun**
![image](https://github.com/user-attachments/assets/db14eb1a-2afc-4791-9d92-8c1d3b5c5bfa)

인텔리제이에서 multirun 플러그인 설치, Run/Debug Configurations에서 스프링부트 & powerShell 로컬 포트 포워딩 연동

## SSH 방식과 비교한 이점
**SSH 터널링을 설정 및 생성하지 않아도 된다.**  
1. ec2 보안그룹 및 역할이 필요 없다.
2. ssh 사용자마다 공개키 생성 및 인증 절차가 없다.
3. EC2 IP를 경유할 필요가 없다.(배스천x)

## 참고
* [SSM 개요](https://docs.aws.amazon.com/ko_kr/systems-manager/latest/userguide/what-is-systems-manager.html)  
* [AWS SSM으로 EC2 인스턴스에 접근하기 (SSH 대체)](https://musma.github.io/2019/11/29/about-aws-ssm.html)  
* [AWS CLI 자격증명 설정](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html)
* [AWS CLI 자격증명 설정-2](https://kimjingo.tistory.com/209)

