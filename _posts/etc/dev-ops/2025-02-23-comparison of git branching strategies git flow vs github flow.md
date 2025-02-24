---
title: Git Branch 전략 비교 - Git Flow vs GitHub Flow
categories: dev-ops
---

# 소개
브랜치 전략이란 여러 개발자들과 함께 프로젝트를 진행할 때, 브랜치 전략 모델을 활용해 효율적으로 작업을 분담하고 관리하는 일종의 워크플로우(work-flow) 방법을 말한다. 여기서 'workflow'란 '작업 절차를 통한 정보나 업무의 흐름'을 의미하며, 즉, 작업 흐름을 뜻한다.  

브랜치 전략은 소프트웨어 개발에서 없어서는 안 될 매우 중요한 요소이다.  

## 브랜치 부재
깃 브랜치를 사용하지 않고 프로젝트를 진행했다고 가정하자.  

우선, 여러 개발자가 동시에 프로젝트를 진행하는 과정에서 코드 충돌과 작업의 혼란이 빈번하게 발생하게 될 것이다. 그리고 
각 개발자가 독립적인 작업을 할 공간이 부족해 버전 관리가 어려워지게 돠고, 그로 인해 기능 개발 중 안정성 문제나 배포 오류가 자주 발생하게 될 것이다. 
또한, 협업의 비효율성으로 팀워크가 방해받고, 테스트와 피드백이 제대로 이루어지지 않아 프로젝트가 매우 복잡하고 비효율적으로 진행된다. 

그렇다면, 브랜치를 사용한다면 어떻게 될까?  

## 브랜치를 통한 원활한 협업과 관리
1. **독립적 작업 & 동시 작업 가능**  
  * 각 개발자는 독립적인 브랜치에서 작업하므로, 서로의 작업에 영향을 미치지 않고, 병렬로 작업할 수 있다.
2. **단위 관리 & 작업 추적**  
   * 기능 개발이나 버그 수정을 브랜치 단위로 관리하여 작업 진행 상황을 명확하게 추적할 수 있다.
   * 각 브랜치의 커밋 내역을 통해 어떤 작업이 이루어졌는지 쉽게 파악할 수 있어 관리가 용이하다.
3. **배포 안정성과 관리 유연성**  
   * 단위별 브랜치는 메인 코드베이스에 영향을 주지 않고, 기능이 완성되면 메인 브랜치에 병합하여 안정적인 상태로 배포할 수 있다.  

## 브랜치 종류
대표적인 브랜치 전략으로는 Git Flow, GitHub Flow, GitLab Flow 등이 있는데, 이 중 Git Flow와 GitHub Flow 
두 모델에 대해서 중점적으로 알아보려고 한다.

## Git Flow
<img width="1098" alt="Image" src="https://github.com/user-attachments/assets/9451683d-f536-49ac-b3f8-91381268498c" />  
- [출처:Git Branch 전략 비교 - Git Flow vs GitHub Flow](https://devocean.sk.com/blog/techBoardDetail.do?ID=165571&boardType=techBlog)  

Git Flow는 프로젝트에서 브랜치의 역할을 명확히 구분하여 효율적인 개발과 배포를 지원하며, 특히 대규모 프로젝트나 복잡한 배포 프로세스에 적합한 모델이다. 
설명만 들었을 때, 굉장히 좋은 모델이라고 생각이 들겠지만, 그러나 현재 트렁크 기반 워크플로우의 등장과 여러 단점으로 인하여 저물어가고 있다.  

### 트렁크 기반 워크플로우(Trunk-Based Development)란?  
TBD는 모든 개발자가 주 브랜치인 '트렁크'에서 작업 후 메인 브랜치에 병합하는 방식으로, DevOps에서 일반적으로 사용되는 개발 모델이며, 그리고 CI/CD을 촉진하여 애자일 개발 환경에 적합하다.  

### Git Flow는 웹 앱 개발에 적합하지 않다.
* 브랜치 관리 규약이 복잡하다.
  * 규약을 익히는 것과 규약으로 인한 관리의 복잡성의 문제
* 브랜치 긴 라이프사이클
  * 서로 다른 두 브랜치에서 독립적으로 작업을 하게 되면, 추후에 병합할 때 Git 상에서 갈등을 해결해야 한다.
    * 프로젝트 규모가 크거나, 오래 작업되었을수록 갈등을 해결하기 어렵다.
  * 오래 유지되어서 변경사항이 많을수록 코드 리뷰에 대한 어려움
    * 코드 퀄리티나 버그에 대한 피드백을 주기 힘들어진다.
  * 배포의 주기가 길어졌다는 것을 의미한다.  
<br>

이러한 단점들을 해결할 모델 방식이 바로 트렁크 기반 워크플로우이다.  

하지만, 그럼에도 불구하고 Git Flow는 많은 팀과 조직에서 채택한 업계 표준의 브랜치 관리 방식이다.  
Git Flow를 배우면, 프로젝트 관리와 버전 관리의 기본 원칙을 이해하게 된다. 이는 팀에서의 협업이나 실제 기업 환경에서 프로젝트를 관리하는 데 중요한 기초가 된다.  

### 종류
* **master**
  * 제품의 안정된 출시 버전을 관리하는 메인 브랜치로, 항상 배포 가능한 상태를 유지
* **develop**
  * 다음 출시 버전을 위한 개발이 이루어지는 브랜치로, 새로운 기능이나 개선 작업이 반영    
* **feature**
  * 특정 기능을 개발하기 위한 브랜치로, 개발이 완료되면 develop 브랜치에 병합  
* **release**
  * 다음 출시 버전을 준비하는 브랜치로, 최종 테스트 및 버그 수정 작업이 진행 및 완료되면 master와 develop에 병합   
* **hotfix**
  * 이미 출시된 제품에서 발생한 긴급한 버그를 수정하는 브랜치로, 수정 후 master와 develop에 병합  

### 구조
Git-Flow는 항상 유지되는 메인 브랜치인 master와 develop 브랜치, 그리고 작업이 완료되면 사라지는 보조 브랜치인 feature, release, hotfix로 구성된다.  

**메인 & 개발(master & develop)**  

![Image](https://github.com/user-attachments/assets/31560bea-a02c-4efd-9c58-6a0ec7880841)  
- [출처: https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)  
<br>

master 브랜치는 항상 배포 가능한 상태만을 관리하는 브랜치를 말한다. 그리고 develop 브랜치는 다음 배포를 위한 개발이 이루어지는 브랜치로, 주로 기능들이 통합되는 역할을 한다.  

평소에는 개발자들이 이 브랜치를 기반으로 작업을 진행하며, 새로운 기능이나 수정 사항을 반영한다.  
<br>

**기능 브랜치(feature)**  

![Image](https://github.com/user-attachments/assets/64b63876-7a4a-43a2-8a8c-f4291e1cd0d9)  
- [출처: https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)  
<br>

보조 브랜치는 피처 브랜치(feature branch) 또는 토픽 브랜치(topic branch)를 말한다.  

master 브랜치에서 develop 브랜치를 만들고, 그 후 develop 브랜치에서 다시 feature 브랜치를 나눠 작업하는 구조를 그림으로 확인할 수 있다.  

develop 브랜치에는 기존에 잘 작동하는 개발 코드가 포함되어 있으며, 보조 브랜치는 새롭게 변경될 개발 코드를 분리해 관리하는 역할을 수행한다. 보조 브랜치는 기능이 완성될 때까지 유지되며, 작업이 끝나면 develop 브랜치로 병합된다. 만약 결과가 좋지 않으면 해당 보조 브랜치는 버려진다. 보조 브랜치는 보통 개발자 개인의 저장소에서만 존재하며, origin에는 푸시하지 않는다.  
<br>

**릴리즈 브랜치(release)**  

![Image](https://github.com/user-attachments/assets/1a32e186-6fc9-4f23-97a8-6a96853d4cce)  
- [출처: https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)  
<br>

배포를 위한 최종적인 버그 수정 등의 개발을 수행하는 브랜치이다.    

develop 브랜치에 버전으로 포함될 기능들이 병합되면, QA를 위해 release 브랜치를 develop 브랜치에서 생성한다. 배포 준비가 완료되면, release 브랜치를 master 브랜치에 병합하고, 출시된 master 브랜치에는 버전 태그(예: v1.0, v0.2)를 추가한다. 

release 브랜치에서 기능을 점검하며 발견된 버그 수정 사항은 develop 브랜치에도 반영해야 하므로, 배포가 완료된 후 develop 브랜치에도 Merge 작업을 진행해야 한다.  
<br>

**핫픽스 브랜치(hotfix)**  

![Image](https://github.com/user-attachments/assets/d938957f-dd3d-434e-9884-1484e9576bec)  
- [출처: https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)  
<br>

핫픽스 브랜치는 배포한 버전에서 긴급하게 수정할 필요가 있을 때 master 브랜치에서 분리하는 브랜치를 말한다.  

hotfix는 보통 급하게 버그를 수정하기 위해 생성되는 브랜치이므로, 버그가 해결되면 대개 제거되는 일회성 브랜치이다.  

release 브랜치가 생성되어 관리되고 있는 상태라면, hotfix 정보를 해당 브랜치에 병합하여, 다음 배포 시 반영이 정상적으로 이루어지도록 해야 한다.  

버그 수정 작업을 하는 동안에도 다른 개발자들은 develop 브랜치에서 계속 작업할 수 있다. 이때, hotfix 브랜치에서 발생한 변경 사항은 develop 브랜치에도 병합하여 문제를 해결해야 한다.  
<br>

### 흐름
개발자는 신규 기능 개발을 위해 develop 브랜치를 기준으로 feature 브랜치를 생성하여 작업을 진행한다. 작업이 완료된 feature 브랜치는 develop 브랜치에 병합되며, 이 과정은 PR(Pull Request)을 통해 작업 내용을 리뷰받고, 리뷰가 완료된 후 PR을 병합하는 방식으로 진행된다.

다음 출시 버전을 준비하기 위해 develop 브랜치에서 release 브랜치를 생성하고 배포 준비를 시작한다. 이 과정에서 발견된 버그는 release 브랜치에서 바로 수정된다. 충분한 테스트 후, 일정한 주기(일반적으로 배포하려는 버전 단위)로 release 브랜치는 master 브랜치에 병합되어 제품이 출시된다.

상용 배포 후, release 브랜치에서 미처 발견되지 않은 새로운 버그는 hotfix 브랜치에서 신속하게 수정 작업을 진행한다.

## GitHub Flow
<img width="1046" alt="Image" src="https://github.com/user-attachments/assets/14578fef-bf02-4e73-9e42-71fa57b782b4" />  
- [출처:Git Branch 전략 비교 - Git Flow vs GitHub Flow](<img width="1046" alt="Image" src="https://github.com/user-attachments/assets/14578fef-bf02-4e73-9e42-71fa57b782b4" />)  

GitHub에서 사용하는 방식인, GitHub Flow는 직관.단순한 브랜치 기반 워크플로로, 작은 변경을 자주 통합하여 빠르고 지속적인 배포를 가능하게 하는 방법이다.  

주 브랜치 master에 기능을 추가하기 위한 브랜치 feature 두 개만을 운용 및 통합하여 더 빠르게 수정, 배포할 수 있는 전략이다.  

### 흐름
1. **브랜치**  
* master 브랜치는 통합 및 배포 목적으로 관리되는 주 브랜치이다.
* feature 브랜치는 master 브랜치를 따서 생성 및 작업을 진행한다.
* feature 브랜치는 기능, 에러, 버그, 리팩토링 등등 여러 목적으로 사용될 수 있다.
  * 해당 브랜치 명과 역할에 대해 자세하게 기입할 것  

2. **커밋 & 푸쉬**  
* 커밋 메시지 자세하게 기입
* 자주 빈번하게 push  

3. **PR(Pull Request)**  
* Review, Merge 상황에 PR을 통해 브랜치를 공유한다.  

4. **리뷰 & 토의**  
* 해당 브랜치에 대해 리뷰와 토의를 진행한다.  

5. **테스트**  
* 통합된 내용을 바탕으로 서버에서 테스트와 배포 테스트를 진행한다.
* 문제 발생 시, 문제 지점을 해결하고, 이를 반복한다.  

6. **Merge**  
* 테스트가 성공적으로 마쳐졌다면, master에 푸쉬 및 배포를 진행한다.  

## 두 가지 전략 비교 및 선택
Git Flow와 GitHub Flow 중 어떤 전략을 선택할지는 프로젝트의 규모, 팀의 요구 사항, 그리고 개발 및 배포 방식에 따라 달라진다.  

### 브랜치 수
* **Git Flow**  
다양한 종류의 브랜치를 사용한다. 주로 master, develop, feature, release, hotfix 등 여러 브랜치를 통해 명확하게 작업을 분리하고 관리한다.  

* **GitHub Flow**  
단일 브랜치(master)만 사용하며, 기능 개발을 위한 feature 브랜치를 따로 관리한다. 복잡성이 적고 단순하다.  

### 배포 방식
* **Git Flow**  
release 브랜치와 hotfix 브랜치를 통해 배포 절차가 명확하고 체계적이다. 이 방식은 각 단계의 안정성을 확인하면서 배포할 수 있다.  

* **GitHub Flow**  
master 브랜치에서 바로 배포를 수행하며, 지속적 배포(CD)를 강조한다. 배포가 빠르고 단순하지만, 배포 과정에서 테스트와 검증이 부족할 수 있다.  

### 복잡성
* **Git Flow**  
복잡한 프로젝트나 대규모 팀에 적합한 전략이다. 많은 브랜치를 사용하고 각 브랜치에 대한 명확한 역할을 정의하여, 보다 체계적인 버전 관리와 배포가 가능하다.  

* **GitHub Flow**  
단순하고 빠른 개발 및 배포를 위해 설계되었다. 불필요한 브랜치나 절차가 없고, 기능을 빠르게 테스트하고 배포할 수 있는 유연성을 제공한다.  

### 유연성 및 안정성
* **Git Flow**  
더 많은 제어와 복잡성을 가지고 있어, 특정 기능이나 수정을 신속하게 배포해야 하는 경우에는 다소 유연성이 떨어질 수 있다. 그러나 안정적인 배포, 버전 관리 및 롤백을 필요로 하는 경우 매우 유용하다.  

* **GitHub Flow**  
단순하고 빠른 배포를 가능하게 하지만, 테스트와 검증 절차 없이 master 브랜치로 바로 병합되므로 잠재적인 위험이 존재한다. 그러나 빠르게 반복 가능한 작업을 원할 때 유리하다.  

### 적합한 환경
* **Git Flow**  
대규모 팀이나 복잡한 프로젝트에 적합하며, 체계적인 배포 관리, 버전 관리 및 롤백을 중요시하는 환경에서 유리하다.  

* **GitHub Flow**  
작은 팀이나, 지속적인 배포가 중요한 프로젝트, Agile 개발 환경이나 테스트와 검증을 신속하게 진행해야 하는 프로젝트에서 유리하다.  
<br>

# 참고
- [Git Branch 전략 비교 - Git Flow vs GitHub Flow](https://devocean.sk.com/blog/techBoardDetail.do?ID=165571&boardType=techBlog)  
- [[GIT] 📈 깃 브랜치 전략 정리 - Github Flow / Git Flow](https://inpa.tistory.com/entry/GIT-%E2%9A%A1%EF%B8%8F-github-flow-git-flow-%F0%9F%93%88-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EC%A0%84%EB%9E%B5)
- [Trunk-based development](https://www.atlassian.com/ko/continuous-delivery/continuous-integration/trunk-based-development)
- [트렁크 기반 개발(Trunk-Based Development) 방식과 그 장점](https://f-lab.kr/insight/trunk-based-development)
- [Gitflow workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [Git Flow에서 트렁크 기반 개발으로 나아가기](https://tech.mfort.co.kr/blog/2022-08-05-trunk-based-development/)