---
title: EC2 스프링 빌드 시 먹통 현상
categories: aws
---

## 이슈
ec2의 ubuntu에서 spring boot 빌드가 먹통이 되는 현상이 발생하였다. 로컬 환경에서는 문제가 없었지만, 환경이 바뀌면서 여러 가지 설정 파일이 문제가 될 거라고 판단이 했기에 금방 해결할 거라고 생각했다. 

## 문제
![issue1](https://github.com/user-attachments/assets/baaed211-9564-4719-9157-12b206144962)

사진과 같이 먹통 현상이 발생하면 아무런 명령어도 입력을 받을 수 없어 사용자가 할 수 있는 거라곤 ec2 인스턴스를 종료 후 다시 재시작을 해야 하는 시간 비용이었다.

**관련 파일들이 원인일 것이다.**  
이 때문에 .env, application.yml, gradle.yml 이 파일들의 내부 요소들이 발목을 잡아 빌드 중에 원인을 알 수 없는 에러가 발생해서 그런 거라고 굳게 믿고 계속해서 끊임없는 반복을 시도하였다. 그리고 5시간 정도 흐르고 이 방법이 잘못됐다는 것을 깨닫게 되었다.

**ec2 프리티어 메모리는 1기가였다.**  
검색을 하며 찾아보니 나와 같은 상황을 미리 격은 여러 사람들이 메모리 부분이 원인이라고 그것에 대한 해결점을 글을 올려두었다. 반신반의했지만, 여차 다른 방법은 없었기에 이 방법을 믿어보기로 결심했다.  

## 스왑 공간
ec2의 메모리는 1기가다. 따라서 빌드 동작 과정에서 충분하지 못하여 끌어다 쓸 자원이 더 이상 없으니 서버는 먹통이 되었다. 이를 해결하기 위해선 가상의 메모리 공간을 만들어 부족할 때 가져다 사용해야 한다.

![issue2](https://github.com/user-attachments/assets/e449e49b-c3c5-44bd-a428-f9a299964e87)

위 내용대로 1GB의 2배 정도의 스왑 공간을 확보하라고 설명되어 있으니, 그것만큼 설정하면 될 것 같다.
```bash
1. $ sudo dd if=/dev/zero of=/swapfile bs=128M count=16
# 스왑 메모리 16 * 128 = 2GB 
1. $ sudo chmod 600 /swapfile
# 스왑 파일의 읽기 및 쓰기 권한 업뎃
1. $ sudo mkswap /swapfile
# Linux 스왑 영역을 설정합니다.
1. $ sudo swapon /swapfile
# 스왑 공간에 스왑 파일을 추가하여 스왑 파일을 즉시 사용할 수 있도록 설정.
1. $ sudo swapon -s
# 절차가 성공적으로 완료되었는지 확인
1. $ sudo vi /etc/fstab
# 부팅 시 /etc/fstab 파일을 편집하여 스왑 파일을 시작
6-1. /swapfile swap swap defaults 0 0
# 파일 끝에 다음 새 줄을 추가하고 파일을 저장한 다음 종료
1. sudo reboot
# 재부팅
```

## 참고
- [https://repost.aws/ko/knowledge-center/ec2-memory-swap-file](https://repost.aws/ko/knowledge-center/ec2-memory-swap-file)
- [https://repost.aws/ko/knowledge-center/ec2-memory-partition-hard-drive](https://repost.aws/ko/knowledge-center/ec2-memory-partition-hard-drive)