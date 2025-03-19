---
title: 3 - private RDS 접근하기
categories: ci-cd-auth-project
---

# AWS RDS MySQL
현재 AWS EC2 인스턴스를 사용하여 서버를 구동하고 있으며, AWS RDS는 MySQL을 지원하는 관리형 데이터베이스 서비스이다.
EC2와 RDS는 동일한 VPC 내 서브넷에서 원활하게 통합되며, 이를 통해 데이터베이스 관리와 운영의 복잡성을 크게 줄일 수 있다. 이러한 이유로 AWS RDS를 채택하게 되었다.  

<br>

## 왜 Private RDS인가?
### VPC Subnet
VPC Subnet은 리소스를 공용 Subnet에 배치하여 외부와 통신이 필요한 public subnet, 외부와 통신할 필요 없이 내부 네트워크에서만 다른 자원들과 통신할 수 있는 private subnet으로 구분할 수 있다. 이렇게 구분함으로써 성능을 최적화하고 보안을 강화할 수 있다.

Private subnet에 있는 리소스들은 인터넷과 직접 연결되지 않기 때문에, 외부의 불필요한 트래픽을 차단할 수 있다. 이로 인해 서버나 데이터베이스가 외부와의 네트워크 통신 없이, VPC 내부 네트워크 내에서만 통신이 이루어지기 때문에 더 빠르고 안정적인 데이터 전송이 가능해진다.  

Private subnet은 외부 네트워크에서 접근할 수 없기 때문에, 중요한 데이터베이스나 애플리케이션 서버는 외부 공격이나 해킹으로부터 보호된다. VPC 내에서 암호화된 전용 네트워크 연결을 통해 데이터가 전송되므로, 외부의 위험 요소를 차단하면서도 안전하게 내부 네트워크 간의 통신을 유지할 수 있다. 또한, Private subnet에 배치된 리소스들은 Security Group과 Network ACL을 사용해 세밀한 접근 제어가 가능하다. 이를 통해 외부에서의 직접적인 접근을 차단하고, 오직 내부 애플리케이션이나 특정 서버만 접근할 수 있도록 설정할 수 있다.  

### Public Ip 과금
AWS는 한 계정당 Public IP 사용 시간을 750시간 무료로 제공한다. 그러나 RDS를 공인 IP로 설정할 경우, 한 계정에 두 개의 Public IP가 할당되게 된다. AWS는 IPv4 주소의 고갈 문제를 우려하고 있으며, 이를 방지하기 위해 프리티어 계정이라 할지라도 두 개의 공인 IP를 사용하면 시간 초과로 과금이 발생할 수 있다. 이러한 이유로, 나는 EC2 외에는 공인 IP를 사용하지 않는다.  

<br>

## SSH Tunneling & Port Forwarding
![Image](https://github.com/user-attachments/assets/2b7d1ded-0dfb-4f7f-bc5f-1e4c4117acea)  

일반적으로 Private RDS는 인터넷에서 직접 접근할 수 없도록 설정되어 있기 때문에, EC2 중간 서버를 통해 우회해서 접근해야 한다. 이 방식을 가능하게 해 주는 방식이 일반적으로 두 가지가 있는데, SSM과 SSH이다.  

SSM은 EC2 인스턴스를 중계 서버로 사용하지 않고도 RDS에 접근할 수 있지만, 설정과 사용 방법이 복잡하고 추가적인 구성 작업이 필요하기 때문에 이를 사용하지 않기로 하였다.  

SSH는 암호화된 연결을 제공하여 외부에서 EC2로 안전하게 접속할 수 있도록 해주며, 이를 통해 RDS에 접근할 수 있다.  

<br>
---

### SSH 접속 WARNING: UNPROTECTED PRIVATE KEY FILE!
![Image](https://github.com/user-attachments/assets/83cbf493-cc6c-432e-9e93-9fa32fcb4b67)  

SSH 프로토콜에 원격 요청을 보냈을 때 위와 같은 에러가 발생했다. 이는, SSH가 보안 프로토콜이고, 개인 키 파일에 너무 많은 사용자가 읽기 또는 쓰기 권한이 남용됐을 경우에 여러 사람이 공용으로 접근할 수 있기 때문이다. 따라서 소유자에게만 읽기 및 쓰기 권한을 부여하고 다른 사용자에게는 모든 권한을 제거해야 한다.  

**리눅스**  
```
chmod 600 /path/to/your/private/key

6: 소유자(owner)에게 읽기(read) 권한(4)과 쓰기(write) 권한(2)을 부여. (4 + 2 = 6)
0: 그룹(group)에게 아무 권한도 부여하지 않음.
0: 다른 사용자(others)에게 아무 권한도 부여하지 않음.
```
리눅스, 유닉스 계열 시스템에서 사용되는 명령어다.  

**윈도우**
```
icacls.exe key-name.pem /reset
-> 권한을 초기화
icacls.exe key-name.pem /grant:r %username%:(R,W)
-> 개인키를 소유한 현재 사용자에게 읽기(R), 쓰기(W) 권한을 부여한다. /grant:r 옵션은 기존 권한을 덮어쓰고 새 권한을 부여하는 옵션
icacls.exe key-name.pem /inheritance:r
-> 개인키의 상속된 권한을 제거한다. 
-> Windows에서는 기본적으로 파일이나 폴더가 부모 폴더에서 권한을 상속을 받는다. 때문에, 의도치 않은 사용자나 프로세스가 해당 파일에 접근할 가능성이 있다.
```
윈도우에서 사용되는 명령어다.  

<br>
---

### Local Port Forwarding
![Image](https://github.com/user-attachments/assets/9e77e99f-84b4-41a0-99ff-36ee3e82c3cf)  

SSH 터널링을 통해 호스트 -> EC2 -> RDS 구조를 구축하려면, 터널링을 설정해야 한다.  

**명령어로 터널링 연결하기**  
```
ssh -i /path/to/your-key.pem -L 3307:rds-endpoint:3306 ec2-user@ec2-public-ip

* /path/to/your-key.pem: EC2 인스턴스에 접근할 수 있는 개인 키 파일.
* rds-endpoint: RDS의 엔드포인트 주소.
* 3306: MySQL 같은 DB 서비스의 기본 포트. (다른 DB라면 해당 포트를 사용)
* ec2-user@ec2-public-ip: EC2 인스턴스의 사용자 이름 및 퍼블릭 IP.
```

<img src="https://github.com/user-attachments/assets/9ec6c076-831f-43f6-8bb4-a9a237d1d3dc" style="width: 50%; height: auto">  

SSH 터널에 정상적으로 접속이 되었다면, 위와 같이 나올 것이다.  

**SpringBoot에서 RDS 인스턴스 접근하기**  
```
    url: jdbc:mysql://RDS 엔드포인트:RDS PORT/RDS 데이터베이스명
    username: RDS 아이디
    password: RDS 비밀번호
```

이제 로컬에서 RDS 인스턴스에 접근할 수 있는 환경이 조성되었다. 다음으로, yml 파일을 설정하여 RDS에 접속할 수 있도록 구성할 것이다.  

![Image](https://github.com/user-attachments/assets/0e29e924-bd5a-46e7-9e36-7fb622570bd6)  

> ERROR 2436 - Communications link failure  

처음에는 RDS 엔드포인트와 DB명을 기입 후에 접속을 시도해 보았다. 그 결과, 연결이 실패한 것을 알 수 있었다.  

```
    url: jdbc:mysql://localhost:3308/local_database_name
    username: your_mysql_username
    password: your_mysql_password
```

![Image](https://github.com/user-attachments/assets/58e1a394-9939-4493-bf74-15e5112f42a7)  

두 번째로는 URL 앞에 localhost:3308, 로컬 데이터베이스의 DB명, 이름, 암호를 기입하여 시도하였다. 로컬 데이터베이스 명을 기입한 이유는 localhost:3308에서 요청을 보내는 것이기 때문에, 로컬 데이터베이스 설정 정보가 기반이 될 줄 알았기 때문이다. 결과는 RDS 데이터베이스와 연결이 된 것이 아닌, 로컬 데이터베이스와 연결이 되었다.  

```
    url: jdbc:mysql://localhost:${LOCAL_DB_PORT}/${RDS_SCHEMA}?useSSL=false&serverTimezone=UTC
    username: ${RDS_USERNAME}
    password: ${RDS_PWD}
```

![Image](https://github.com/user-attachments/assets/273191a3-c2e9-4e0d-9a8d-97193c0d9de5)  

세 번째, 마지막 시도에서는 로컬 DB명이 아니라 RDS DB명, 아이디, 비밀번호를 사용했다. 이 방법이 성공할 수 있었던 이유는 SSH 터널링 이론을 다시 한 번 자세히 공부하고, SSH 터널링 명령어를 한 번 더 살펴본 덕분이었다. 터널링이 연결된 상태에서, 클라이언트가 RDS로 접근할 수 있는 경로가 마련되었기 때문에, 로컬 주소와 포트를 시작으로 RDS DB 주소로 접속하는 방식으로 이해하면 된다.  

<br>
---

**application.yml 파일에서 SSH 터널 설정하기**  
일일이 터널링을 생성해서 RDS에 접근하는 것은 번거롭고 시간이 소모되는 작업이다. 이를 개선하기 위해 JSCH 라이브러리를 활용하여 SSH 연결을 자동화하려고 한다.  

JSCH (Java Secure Channel)는 Java에서 SSH(Secure Shell) 연결을 관리하고 사용할 수 있도록 도와주는 라이브러리로서, 주로 원격 시스템에 안전하게 연결하고 파일을 전송하거나 명령을 실행할 때 사용된다. 현재는 오리지널 jcraft의 JSch는 더 이상 지원되지 않기 때문에, mwiede의 JSch 라이브러리를 사용하고 있다.  

크게 네 단계를 나눠 각각을 구성 및 설정해 주어야 한다.  
```
--- build.gradle ---

dependencies {
	implementation 'com.github.mwiede:jsch:0.2.24'
}
```

mwiede:jsch 라이브러리 설치  

```
--- application.yml ---

spring:
  ssh:
#   원격 서버에 연결하기 위한 중계 서버를 지정하는 부분
    remote_jump_host: ${SSH_HOST}
#   SSH 연결에 사용할 사용자 이름
    user: ${SSH_USER}
#   SSH 연결에 사용할 포트 번호
    ssh_port: 22
#   SSH 연결을 위한 private key(개인 키)의 경로 or 값을 지정
    private_key: ${SSH_PRIVATE_KEY}
#   실제 데이터베이스 서버의 엔드포인트 URL
    database_url: ${RDS_ENDPOINT}
#   실제 데이터베이스 포트 번호
    database_port: ${RDS_PORT}
```

위와 같이 application.yml 파일에서 설정을 구성한다.  

```
--- config.SshTunnelingConfig ---

import com.jcraft.jsch.JSch;
import com.jcraft.jsch.JSchException;
import com.jcraft.jsch.Session;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import jakarta.annotation.PreDestroy;
import java.util.Properties;

@Component
public class SshTunnelingConfig {

    private static final Logger log = LoggerFactory.getLogger(SshTunnelingConfig.class);

    private Session sshSession;

    // application.yml에 구성된 특정 로컬 포트 사용
    @Value("${LOCAL_DB_PORT}")
    private int localPort;

    @Value("${spring.ssh.remote_jump_host}")
    private String remoteHost;

    @Value("${spring.ssh.user}")
    private String sshUser;

    @Value("${spring.ssh.ssh_port}")
    private int sshPort;

    @Value("${spring.ssh.private_key}")
    private String privateKeyPath;

    @Value("${spring.ssh.database_url}")
    private String databaseHost;

    @Value("${spring.ssh.database_port}")
    private int databasePort;

    /**
     * SSH 터널링을 설정하고 로컬 포워딩 포트를 반환합니다.
     *
     * @return 로컬에서 포워딩된 포트 번호
     * @throws JSchException SSH 연결 중 문제가 발생한 경우
     */
    public int setupSshTunnel() throws JSchException {
        JSch jsch = new JSch();

        String formattedKey = privateKeyPath.replace("\\n", "\n");
        jsch.addIdentity("sshKey", formattedKey.getBytes(), null, null);

        log.info("SSH 연결 설정: {}:{} (사용자: {})", remoteHost, sshPort, sshUser);
        Session session = jsch.getSession(sshUser, remoteHost, sshPort);

        // 엄격한 호스트 키 검사 비활성화
        Properties config = new Properties();
        config.put("StrictHostKeyChecking", "no");
        session.setConfig(config);

        log.info("SSH 서버에 연결 중...");
        session.connect(30000); // 30초 타임아웃

        // 나중에 정리를 위해 세션 저장
        this.sshSession = session;

        log.info("포트 포워딩 설정: localhost:{} -> {}:{}", localPort, databaseHost, databasePort);
        session.setPortForwardingL(localPort, databaseHost, databasePort);

        log.info("SSH 터널이 로컬 포트 {}에 성공적으로 설정되었습니다", localPort);
        return localPort;
    }

    /**
     * 애플리케이션 종료 시 SSH 세션을 정리합니다.
     */
    @PreDestroy
    public void closeSSH() {
        if (sshSession != null && sshSession.isConnected()) {
            log.info("SSH 터널 닫는 중...");
            sshSession.disconnect();
            log.info("SSH 터널이 닫혔습니다");
        }
    }
}
```

이 코드는 JSch 라이브러리를 활용하여 SSH 터널을 생성하고, 로컬 포트와 원격 데이터베이스 포트 간에 암호화된 연결을 설정한다.

```
--- config.DataSourceConfig ---

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

import javax.sql.DataSource;

@Configuration
public class DataSourceConfig {

    private static final Logger log = LoggerFactory.getLogger(DataSourceConfig.class);

    private final SshTunnelingConfig sshTunnelingConfig;

    @Value("${RDS_SCHEMA}")
    private String schemaName;

    /**
     * 생성자를 통한 의존성 주입
     *
     * @param sshTunnelingConfig SSH 터널링 설정 객체
     */
    public DataSourceConfig(SshTunnelingConfig sshTunnelingConfig) {
        this.sshTunnelingConfig = sshTunnelingConfig;
    }

    /**
     * SSH 터널링을 통해 접근 가능한 데이터소스를 구성합니다.
     *
     * @param properties 데이터소스 속성
     * @return 구성된 DataSource 객체
     */
    @Bean
    @Primary
    public DataSource dataSource(DataSourceProperties properties) {
        try {
            // SSH 터널 설정 및 로컬로 포워딩된 포트 가져오기
            int forwardedPort = sshTunnelingConfig.setupSshTunnel();

            // 올바른 포워딩 포트로 JDBC URL 구성
            String jdbcUrl = String.format("jdbc:mysql://localhost:%d/%s?useSSL=false&serverTimezone=UTC",
                    forwardedPort, schemaName);

            log.info("SSH 터널을 통한 JDBC URL 생성: {}", jdbcUrl);

            // DataSource 구성 및 반환
            return DataSourceBuilder.create()
                    .url(jdbcUrl)
                    .username(properties.getUsername())
                    .password(properties.getPassword())
                    .driverClassName(properties.getDriverClassName())
                    .build();
        } catch (Exception e) {
            log.error("데이터베이스 연결을 위한 SSH 터널 설정 실패", e);
            throw new RuntimeException("데이터베이스 연결 실패: " + e.getMessage(), e);
        }
    }
}
```

위에서 생성한 SSH 터널을 통해 로컬 포트로 전달되는 연결을 사용하여 데이터베이스에 접속하는 DataSource를 구성한다.  

<br>
---

**SSH Private Key 값을 사용해서 SSH 터널 생성하기**  
```
// SSH 키는 여러 줄 형식이어야 제대로 작동하므로, 이 코드는 원래의 줄바꿈 형식을 복원하는 역할
// 문자열 내에서 "\n" 문자열을 찾아 실제 줄바꿈 문자인 "\n"으로 대체
    String formattedKey = privateKeyContent.replace("\\n", "\n");
    jsch.addIdentity("sshKey", formattedKey.getBytes(), null, null);
```

기존에는 private key의 경로를 기입하는 방식으로 사용했지만, CI/CD 작업을 병행하는 경우에는 개인키를 복사해야 하는 상황이 발생할 수 있다. 이 방식은 여전히 사용해도 문제가 없지만, 개인키를 복사하는 것은 바람직하지 않다. 왜냐하면 개인키는 해당 키의 유일한 소유자만이 소유할 수 있기 때문이다. 따라서 개인키를 복사해서 사용하는 대신, 가져오는 방식을 고려해야 한다.  

예를 들어, A 클라이언트의 개인키를 가상 머신에서 사용하려면, 개인키를 복사하는 것이 아니라 가상 머신으로 가져와야 한다. 이때, 개인키가 여러 줄로 띄어쓰기가 포함되어 있을 수 있으므로, 이를 하나의 연속된 데이터로 변환해야 한다. 변환 후에는 다시 원래 형식으로 되돌려 사용하는 과정이 필요하다.



