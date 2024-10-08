---
title: Gradle CI MySQL 워크플로우 설정 및 연동 중 문제 해결
categories: github_actions
---

CI 과정에서 빌드 테스트가 왜 필요한지 의문이 들었다. 왜냐하면 로컬 상에서 빌드 처리에 문제가 없다면 서버에 통합을 요청해도 된다고 생각했기 때문이다. 하지만 이전에 배포했었던 AWS RDS와 MySQL 데이터베이스 연동이 JPA 테이블 작명 규칙을 지키지 않아 연동이 되지 않는 이슈를 겪고 나서 생각지도 못한 다양한 변수들이 있기 때문에 이번 CI/CD 과정에서 빌드 이슈를 차라리 빨리 만나서 고생을 하면 좋겠다고 내심 바랬고, 이게 웬걸 즉각적으로 그 소원을 ..... 이룰 수 있게 되었다.

## Gradle CI Database(MySQL) 연동 이슈  
![image](https://github.com/user-attachments/assets/4f2648c1-dff8-4a4a-9093-d330e1922354)

로컬 상에서 빌드 테스트에 이상이 없었고, 바로 서버에 CI 요청을 명령했다. 그랬더니 위와 같은 오류가 발생했다. 

## 문제는 데이터베이스 구축
해당 로그를 가지고 분석을 해보니 여러 가지 조건이 겹쳐서 빌드 테스트 과정에서 문제가 발생한 것으로 보였다. 생각하기에 조건 중 데이터베이스 부분도 포함되어 있었기에 아마 서버에 따로 설정을 해둬야 하는구나 싶었다.

## 서버용 db 환경 구축하기
**Github marketplace**
![image](https://github.com/user-attachments/assets/44428f49-2d6e-456c-88c2-8826dd160288)
깃허브에서는 남이 만든 actions의 동작들을 라이브러리처럼 사용할 수 있게 별도의 장소를 제공한다. 여기서 mysql의 코드를 복사해서 yml에 통합시키면 된다.

## 이번엔 로컬에서..?
해결되는가 싶었지만... 똑같이 contextLoads() FAILED... 오류가 발생했고 계속 만져보면서 알게된 점은 로컬 빌드 과정 또한 Actions Workflow에서는 문제가 없어야 했다. 즉 로컬, 서버 두 세팅이 전부 통과가 되어야만 했다.

**로컬 DB -> 서버 DB와 연동**
```yaml
# workflows의 MySQL 환경설정 코드
 - name: Setup MySQL
        uses: mirromutth/mysql-action@v1.1
        with:
         host port: 3306                      # 애플리케이션에서 사용하는 포트
         container port: 3306                 # MySQL 컨테이너의 포트
         mysql version: '8.0'                 # MySQL 버전
         mysql database: 'giwoot_jang_schema' # 사용 중인 데이터베이스 이름
         mysql root password: ${{ secrets.MYSQL_ROOT_PASSWORD }} # 'root' 사용자 비밀번호를 GitHub Secrets에 설정
         mysql user: ''                       # 비워두면 root 사용자 사용
         mysql password: ''                   # 비워두면 root 사용자 비밀번호 사용

# local의 application.yml의 db 코드
    spring:
    config:
        import: optional:file:.env[.properties]

    datasource:
        url: jdbc:mysql://localhost:3306/giwoot_jang_schema
        username: root
        password: ${MYSQL_ROOT_PASSWORD}
        driver-class-name: com.mysql.cj.jdbc.Driver
    jpa:
        hibernate:
        ddl-auto: update
        show-sql: true
        properties:
        hibernate:
            format_sql: true
```

로컬 상에서 빌드 테스트가 문제없었다면 해당 코드를 서버 DB 코드에 이식을 시켜야 한다. 해당 과정에서 연동도 중요하지만 가장 신경써야 할 부분은 **보안 문제**다. 로컬 상에서 계정 정보는 외부 환경에서 가져와 감추고 이를 hub에 저장할 때 ignore 시켜 보안을 신경쓰게 되는데, 서버 측 또한 actions의 secrets 기능을 활용해서 똑같이 감춰야 한다.

**.env파일을 yml에 참조시키기**  
.env는 로컬 상에서의 환경설정 데이터를 저장하는 용도로 쓰이는 파일이다. 안에 내용물은 보통 KEY='' 와 같은 형식으로 작성하면 secrets 데이터처럼 등록할 수 있다. 
```yml
# application.yml code....
# 데이터베이스
spring:
  config:
    import: optional:file:.env[.properties]

  datasource:
    url: jdbc:mysql://localhost:3306/giwoot_jang_schema
    username: root
    password: ${MYSQL_ROOT_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true

# 시큐리티
  security:
    user:
      name: ${SECURITY_NAME}
      password: ${SECURITY_PASSWORD}
```
.env를 yml에 심어주기 위해 "import: optional:file:.env[.properties]" 코드를 삽입하고 이때 가장 중요한 것은 .env 파일은 반드시 폴더 최상단에 배치가 되어야 하고(.gitignore가 있는 곳) 해당 파일은 외부에 노출시켜서는 안되니 .gitignore에 .env를 넣어 제외시켜야 한다. 그리고 외부 데이터를 참조시키기 위해서는 ${}안에 key 명을 넣으면 된다.

```yml
# .env 파일 출력 명령어

## window cmd
type .env

## linux
# .env 파일 로드
source .env
# 변수 출력
echo "MY_VARIABLE: $MY_VARIABLE"
echo "ANOTHER_VARIABLE: $ANOTHER_VARIABLE"
```

**서버 인스턴스에 DB 연동하기**  
```yml
      - name: Create .env file
        run: |
          echo "MYSQL_ROOT_PASSWORD=${{ secrets.MYSQL_ROOT_PASSWORD }}" >> .env
          echo "SECURITY_NAME=${{ secrets.SECURITY_NAME }}" >> .env
          echo "SECURITY_PASSWORD=${{ secrets.SECURITY_PASSWORD }}" >> .env

      - name: Setup MySQL
        uses: mirromutth/mysql-action@v1.1
        with:
          host port: 3306                      # 애플리케이션에서 사용하는 포트
          container port: 3306                 # MySQL 컨테이너의 포트
          mysql version: '8.0'                 # MySQL 버전
          mysql database: 'giwoot_jang_schema' # 사용 중인 데이터베이스 이름
          mysql root password: ${{ secrets.MYSQL_ROOT_PASSWORD }} # 'root' 사용자 비밀번호를 GitHub Secrets에 설정
          mysql user: ''                       # 비워두면 root 사용자 사용
          mysql password: ''                   # 비워두면 root 사용자 비밀번호 사용
```

application.yml 등록이 끝났다면 이제 actions workflow에도 연동을 해줄 차례다. 위와 같이 setup mysql 코드를 local의 db 코드와 mapping 시키면 된다.
단 여기서는 actions secret 데이터를 참조해야 하는데..  
![image](https://github.com/user-attachments/assets/9a3da204-fd1b-49a2-8ee5-bba5a906e9ea)

사진과 같이 key-value 구조의 데이터를 등록하여 workflows ${{}} 코드 안에 심으면 된다.   
**Create .env file** <- 이건 왜 필요한 것일까?  
workflow의 빌드 구동 중 repository에 저장된 application.yml도 DB 설정을 위해선 같이 저장을 해주어 빌드 구동이 같이 되게끔 해주어야 하고, 그리고 이때 
local 상에서 저장된 .env의 데이터들을 서버의 application.yml에서 사용할 수 없기 때문에 secrets에 가져와 이를 기존에 사용하던 .env 변수명에 대입시켜 변수 값으로 문제가 될 상황을 만들지 않게 했다.

