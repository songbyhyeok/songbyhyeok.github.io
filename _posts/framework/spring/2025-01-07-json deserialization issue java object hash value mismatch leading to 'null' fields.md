---
title: JSON 역직렬화 시 Java 객체 해시값 불일치로 일부 필드 값 "null" 처리 문제
categories: spring
---

# 개요
프로젝트 진행 중 역직렬화 과정에서 일부 필드가 "null"로 처리되는 현상이 발생했다. 처음엔 Json, 객체 간 필드명의 불일치로 매핑되지 않는다고 판단했으나, 그게 아니었고 Jackson, Lombok, Java Beans 네이밍 규약에 따른 불일치 문제였다.

# 이슈
![image](https://github.com/user-attachments/assets/82414af5-9f57-49ec-851d-7f3365215618)  
![image](https://github.com/user-attachments/assets/5085a983-06f3-4032-8120-1f81871a369b)  

개인 프로젝트 휴대폰 인증 기능 구현 중, 비동기 방식으로 개인정보(name, fPhoneN, mPhoneN, bPhoneN)를 Json 객체로 보냈을 때, 서버에서 생성된 DTO 객체 내부 필드 "name"는 Json의 키와 정상적으로 매핑되어 값이 할당되었지만, 카멜 표기법을 사용한 다른 필드들(fPhoneN, mPhoneN, bPhoneN)은 "null"로 처리되는 현상을 포착하였다.  
 
# 문제 정의
```JavaScript
-Client-
const smsAuthRequest = {
                        name: name.value,
                        fPhoneN: fPhoneN.value,
                        mPhoneN: mPhoneN.value,
                        bPhoneN: bPhoneN.value,
                    };
```

```Java
-Server-
@Getter
@ToString
public class SmsAuthRequest {
    private String name;
    private String fPhoneN;
    private String mPhoneN;
    private String bPhoneN;
}
```

"name" 객체는 정상적으로 매핑이 되고, 나머지는 카멜 표기법을 사용하고 있다.  
코드 상에서 Json 데이터의 키는 카멜 표기법을 사용하고, DTO 객체의 필드도 동일하게 카멜 표기법을 따르기 때문에 문제가 될 것으로 보이진 않는다. 그렇다면 왜 ObjectMapper가 이들을 올바르게 매핑하지 못하는 것일까?

## 네이밍 규칙에 따른 필드 검증 과정
![image](https://github.com/user-attachments/assets/b48e3d17-41ca-4671-a54e-743e9ebe3e89)

기본적으로 Jackson는 역직렬화 과정에서 setter와 기본 생성자를 통해 검증 목적의 Json key를 생성하는데, 만약 setter가 없다면 getter와 리플렉션을 활용한다. 이 과정에서는 빈 네이밍 규칙에 따라 프로퍼티 메소드 명이 변경될 수 있으며, 이후 변환된 프로퍼티 명을 다시 @JacksonNaming 규칙에 따라 다시 한 번 더 변환 과정이 일어난다. 그렇게 마지막으로 변환된 값을 Json key에 저장 후, Json 객체와 일치하는지 검증 이후 Java 필드 값에 저장된다.

### BeanNaming
* 모든 케이스에서는 가장 맨 앞 하나만 소문자로 변경한다.
* 예외 케이스, 맨 앞 글자가 대문자이면서 그 다음 글자도 대문자인 이어진 대문자 형식이라면 변환되지 않는다. 

### JacksonNaming
@JsonNaming 전략에 따라 네이밍 규칙이 적용된다. 따로 설정하지 않았다면, 기본값인 LOWER_CAMEL_CASE (예: lowerCamelCase ) 에 따라 역직렬화가 동작된다.  
* 모든 케이스에서는 가장 맨 앞 하나만 소문자로 변경한다.
* 예외 케이스, 맨 앞 글자가 대문자이면서 그 다음 글자도 대문자인 이어진 대문자 형식이라면 모두 소문자로 변경된다.

### 생성규칙 예시
![image](https://github.com/user-attachments/assets/582f2a60-4676-43f4-9da9-af8e94621529)

## 매핑되지 못한 이유
![image](https://github.com/user-attachments/assets/49fa9419-8b6f-40a9-a70a-eff32329f8cf)  

JS to Json = {fPhoneN, mPhoneN, bPhoneN}은 역직렬화 과정에서 네이밍 규칙에 따라 Json keys = {fphoneN, mphoneN, bphoneN}으로 변경되는데, JS to Json 객체와 비교 시 변수명이 일치하지 않아 매핑되지 못했던 것이였다.

# 해결 방법
상황에 맞게 네이밍 전략을 잘짜는 것이 중요하다.  
1. Lombok과 Json네이밍 규칙을 고려한 변수명 생성
2. Getter/Setter 임의 생성
   아래의 @JsonProperty와 같다.
3. public 변수
   정보은닉 위배
4. @JsonProperty
   모든 변수명마다 직접 매핑할 프로퍼티명을 기입하는 방법
5. @JsonIgnore
6. @JsonNaming 전략 변경
    KEBAB_CASE : 이름 요소는 하이픈으로 구분됩니다(예: kebab-case ) .
    LOWER_CASE : 모든 문자는 구분 기호 없이 소문자입니다. 예: 소문자 .
    SNAKE_CASE : 모든 문자는 소문자이며 이름 요소 사이에는 밑줄을 구분자로 사용합니다(예: snake_case) .
    UPPER_CAMEL_CASE : 첫 번째 요소를 포함한 모든 이름 요소는 대문자로 시작하고 그 뒤에 소문자가 오며 구분 기호가 없습니다(예:  UpperCamelCase)(default 전략)      
7. 등등...

# 결국엔 Lombok과 Jackson 네이밍 규칙을 고려한 변수명 생성 방식을 사용
```JavaScript
-Client-
const smsAuthRequest = {
                        name: name.value,
                        fPhoneN: fPhoneN.value,
                        mPhoneN: mPhoneN.value,
                        bPhoneN: bPhoneN.value,
                    };
```
JS 필드의 phone이 들어간 변수명 뒤에 N(= number)를 빼기로 결정하였다. 그 이유는 Number 뜻을 빼더라도 휴대폰 번호 자리란 것을 알 수 있고, 그렇게 함으로써 네이밍 규칙을 고려한 객체 생성이 가능하기 때문이다.  

## 다양한 방식 중 기존 방식을 선택한 이유
현재 사용 방식 외에 다른 5가지 방법을 고려했을 때, 각각의 방식에는 다음과 같은 단점이 있다. 
* **getter/setter, @JsonProperty**  
개발자가 일일이 지정해야 하므로 번거로움이 존재하고, 이는 생산성을 저해할 수 있다.
* **public 변수**  
은닉 원칙을 위배  
* **@JsonIgnore**  
변수명을 숨기고 @JsonProperty의 이름을 사용하게끔 유도하는 방식, 추가적인 번거로움을 초래
* **@JsonNaming**  
1. 네이밍 규칙을 변경할 수 있는 방법 중 가장 나은 선택
2. 현재 환경에서는 네이밍 규칙을 변경 x
3. 협업 상황에서는 고려해 볼 가치 o

# 결론
지금까지 JavaScript에서 JSON 데이터를 Java 객체로 역직렬화하는 과정에서 일부 필드가 "null" 값으로 처리되는 현상의 구조적 원인과 해결 방법을 다루어보았다. 처음에는 단순한 매핑 문제로 생각했지만, 실제로는 여러 객체가 오고 가며 복잡한 형식에 따라 데이터가 변환된다는 사실을 깨닫게 되었다. 덕분에 예상보다 더 깊은 이해가 필요했고, 정신적으로 꽤나 힘든 날이었지만 유익한 경험이었다.

# 참고
- [More Jackson Annotations](https://www.baeldung.com/jackson-advanced-annotations)
- [[Java] 불변객체(Immutable Object)의 JSON 직렬화 및 역직렬화](https://shanepark.tistory.com/437)
- [[Lombok][Jackson] Naming convention for getters/setters in Java](https://yungenie.tistory.com/16)
