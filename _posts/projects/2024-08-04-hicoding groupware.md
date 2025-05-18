---
title: hicoding groupware
categories: projects
---

# Hicoding Groupware
<em>클라우드 기반의 그룹웨어 시스템</em>

**깃허브 링크**: <a href="https://github.com/songbyhyeok/hicoding-groupware" target="_blank" style="color: blue; text-decoration: none;">github link</a>  
**개발 인원**: 5명, 팀원  
**기술 스택**:  
- JPA(DataJPA, Criteria)
- RESTFul
- Deploy(AWS(EC2, S3, RDS), Docker, Ubuntu)
- MySQL
- React

<br>

# 기술적 이슈 및 해결과정
 
## 답글, 대댓글 구현(Tree 형태로 재구축 후 DFS-Stack 처리)
![Untitled (5)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/8346c973-435a-42f9-9ea9-cd2ba28eddf4)
<em> 데이터를 계층구조 변환 후 페이징화 처리 과정과, DFS의 두 가지 방식으로 처리했을 때 상황을 결과물로 표현 </em>  

### 출력 과정에서 구조적 이슈
![Untitled (6)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/f6d06b00-a520-4886-8ff6-f879226b33b0)
<em> dfs 출력 방식에 따라 구조가 달라지는 것을 알 수 있다. </em>  

- 레벨 0의 부모들을 게시판에 배치하기 위해 dfs 재귀 방식을 사용했을 때 등록일자 순으로 게시판 상단에서 하단으로 출력되는 문제가 발생했다.
- 이는 등록일자를 역순으로 재구조화 시 간단히 해결될 문제라 판단하여 역출력을 시도하였다. 그러나 위 그림의 reverse order DFS - recursive 형식처럼 depth 0의 배치만 역으로 바뀌는 현상이 발생하였다. 
- 문제는 부모의 자식을 먼저 출력시켜야 되는 부분에 있어 재귀 방식은 처리할 수 없어, 이를 다른 stack 방식을 채택하여 역순의 자식들을 먼저 삽입하고, 지정된 크기 5 이상이 될 경우 출력 및 비우기를 반복하여 해결하였다.

### 계층구조 변환
![Untitled (8)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/c4ef0f6a-0faf-4651-afc8-a09ea452bab0)
<em> "PostReadResponseList"에서 저장되는 객체는 레벨 0에 있는 "Parent" 객체만 포함하고 있으며, 자식들은 "Parent" 객체의 필드인 리스트에 저장된다. </em>  

- 취합된 데이터를 답글이나 대댓글의 형식으로 처리하기 위해 트리 구조로 표현하는 게 적합 및 판단하였고, 동일 레벨의 여러 개 자식을 구성할 수 있는 m-ary 트리 형태로 변환하기로 결정했다.
- 이를 위해 먼저 데이터 집합에서 레벨이 가장 낮은 부모를 최초 리스트에 저장하고 그 외의 자식들은 해당 부모의 리스트에 저장하는 방식으로 처리했다.

### 계층구조화 변환 과정에서 양방향 Entity JSON 변환 시 순환 참조 문제
![Untitled (10)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/f5e8df72-9726-4343-b78f-44e96598a888)
<em> 양방향 매핑 과정에서의 참조 이슈에 대한 오류 분석과 해결 방식으로 Entity가 아닌 DTO 필드로 대체한 것을 나타낸 그림 </em>
  
- 양방향 JSON 변환 과정에서 Stack Overflow 발생했다. 원인은 멤버 필드 Entity 참조 과정에서 부모와 자식의 사이클 형성이 문제였다.
- 이를 방지하기 위해 Entity가 아닌 DTO 필드로 변환시켜 해결했다.

### 페이징 처리 문제
![Untitled (12)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/1a5b04e8-d3bb-41bc-bed7-6aea17f95e56)
<em> Pageable → Page<T> → Pagination 세 단계 절차를 CustomPagination 객체로 대체 </em>  

- 쿼리 결과를 JPA의 Page<T> 객체로 변환하는 과정에서 계층구조에 맞게 가공된 데이터에는 PagingButtonInfo를 적용할 수 없는 문제가 발생했다.
- 이를 해결하기 위해 기존 방식을 CustomPagination 객체에 옮겨 해결했다.

<br>

## 조회수. 좋아요 구현하기
![Untitled (13)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/c026dfed-b767-4e88-ba73-db3f585791ca)
<em> 게시글의 조회수와 좋아요 기능이 작동되고 있다. </em>

![Untitled (15)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/08d0d1e2-10e6-4cb2-802f-a11d6fba98f9)
<em> 좌측은 물리데이터와 카디널리티 관계 정의를 나타내고 있고, 우측은 각 사용자가 조회수나 좋아요 클릭 시 작동되는 Logic을 보여주고 있다. </em>  

* 게시글을 읽거나 좋아요 버튼 클릭 시 횟수가 누적되게 구현하기 
  * 각 사용자마다 개별적으로 동작시키게 하기 위해 물리 모델링에서 게시글과 사용자 사이에 연결되는 기록을 만들어 카디널리티 관계 재구성하기. 
  * 프로세스 처리 과정에서 방문한 적이 있는지, 아닌지 두 가지 여부로 판단하기.
  * 첫 방문자의 경우 최초로 게시글을 읽거나 클릭 시점에 방문자의 정보가 없으므로 동작하도록 처리하기.
  * 이미 방문한 적이 있는 경우는 현재 시간과 방문 시점의 차를 계산하여 설정된 시간을 넘었을 때만 횟수가 누적되도록 설계하기.

<br>

## 사원 생성
![Untitled (16)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/7ef9c291-9d4b-45d6-ae9c-40861b5337db)
부서별 사원 ID와 임시 비밀번호를 제공하는 사원 생성 시스템과 옵션 선택 항목 기준으로 동적 쿼리 조회가 가능한 JPA Criteria JPQL 빌더 클래스 기능을 구현하였다.

### 사원 생성 알고리즘
1. 회원가입 과정에 필요한 부서별 사용자 식별 ID 생성 시스템을 구현하기로 결정했는데, 기능을 개발하기 위해 부서별 번호 체계 정보를 참고하였다.
- 위 그림에서 (BB 02)는 권한에 맞게 ENUM 값을 활용하였고, (ex ADMIN = 01, STAFF = 02) CC는 최대 입사자 수와 현재 입사자 수를 비교하여 계산하였다. (ex if (MAX = 1000, CurrNo = 55) then Max 0의 개수인 3 - CurrNo 0의 개수인 1의 결과인 2개가 CurrNo 앞에 붙여 결과는 ID = 0055가 된다.) 
- 이를 모두 결합하여 사번 ID를 생성하면 다음과 같다. ‘hc + AA(23) + BB(02) + CC(0055) = hc23020055’

2. 해당 알고리즘을 구현하며 고려사항 두 가지는 다음과 같다. 1. 같은 부서에 Next ID 생성, 2. 부서별 최대 인원수를 초과하지 않고 좌측에 여백이 있을 경우 0으로 채우기. 
- 첫 번째 경우는 현재 부서의 마지막 ID + 1 계산 방식을 사용하여 처리하기.
- 두 번째 경우는 좌측에 0이 들어갔을 경우에 대해 (ex Max = 1000, CurrVal = 100일 경우), 결과 값은 ‘0100’로 가정 및 접근하여 Max와 CurrVal의 0의 자리를 뺀 값을 조합하여 문제를 해결하기.

<br>

## 사원 조회
- 기존 Spring Data JPA로 복잡한 JQPL을 처리 문제에 대해 JPA Criteria를 혼용하여 동적 쿼리 조회 기능을 구현 중 두 가지 애로사항이 발생하였다.
- Criteria 빌더 재구조화 문제와 조회 이후 Front에서 정렬 처리 두 가지였다.

### 해결 과정
![Untitled (19)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/0c824781-813f-4cd7-b4c4-01d4fb6cd9b6)
- 하나의 쿼리를 호출하기 위해서는 복잡한 절차와 상당한 비용이 들 것임을 인식하였고, 방치하게 된다면 유지보수가 어려워질 것으로 판단하였다. 그래서 이를 해결하기 위해 리팩토링을 통해 해결하였다.
- 조회 후에도 정렬을 위해 Criteria 정렬 쿼리를 요청하는 것은 비용이 높다고 판단되어 사용하지 않고, Front 자체에서 JS 정렬 처리 알고리즘을 사용하여 문제를 해결하였다.

<br>

## 게시판과 댓글 CRUD 구현 (React(Redux, Thunk), RestAPI CRUD)
![Untitled (20)](https://github.com/songbyhyeok/2023-HicodingGroupware/assets/63230518/85913404-be35-4b14-bb45-296833d4678a)
<em> React Redux와 Spring Restful을 결합하여 구현된 게시판과 댓글 CRUD 그림 </em>  

- 주어진 두 주기인 React의 Redux와 Thunk, 그리고 Spring과 Restful API 시스템을 연동하여 게시판과 댓글 CRUD 기능을 구현하였다. 
- 클라이언트 측에서는 Redux를 사용하여 웹의 데이터 상태를 저장 및 관리하고, 그리고 Thunk는 Dispatch 역할 및 비동기 작업 처리를 담당한다. 
- Restful API는 양 측 간의 데이터 교환 및 표준화 역할을 수행한다.


