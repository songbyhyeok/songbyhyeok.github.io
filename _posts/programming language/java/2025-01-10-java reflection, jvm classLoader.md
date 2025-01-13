---
title: Java Reflection, JVM ClassLoader
categories: java
---

# 서론
"지나가던 길, 근사한 아파트가 눈에 띈다. 스마트폰 어플을 실행하고, 내가 마음에 드는 동과 호수를 선택한다. 그 순간, 그 아파트의 명의가 내 것으로 바뀐다. 마치 프로그램 코드 한 줄로 세상을 조작하는 것처럼, 내가 원하는 대로 모든 것이 변한다. 이건 정말 현실일까?"  

우리는 종종 세상이 내가 생각하는 대로, 마법처럼 바뀌기를 꿈꾼다. 실제로 그런 능력은 없는... 아니, AI라면 이런 일이 현실처럼 가능해질지도 모른다. 자, 그렇다면 SW 세상에서는 이런 일이 빈번히 발생한다면 어떨까? 바로 **리플렉션**이라는 기술 덕분에 말이다.  
<br>

# Reflection
![javalang](https://github.com/user-attachments/assets/6425d87e-1bdb-433e-b7bd-9c92698063d4)
![reflection](https://github.com/user-attachments/assets/05b76a99-6969-4aa4-a051-c4b7b9ce970e)  
출처: [https://www.geeksforgeeks.org/reflection-in-java](https://www.geeksforgeeks.org/reflection-in-java)  

리플렉션은 클래스 로더에서 읽어들인 클래스들을 바탕으로 런타임 때, 객체 정보를 활용해서 클래스를 분석하고 조작하는 Java 기법API다.  

## 어떻게 분석하고 조작하는 것일까? 
런타임 시점에 클래스가 존재하고 객체명을 알고 있다면, 그 클래스의 메타데이터인 메서드, 타입, 변수들에 접근 및 조작이 가능하다. 좀 더 구체적으로 호출.조회.생성.수정과 같은 행위들을 할 수 있게 된다. 
주로 프레임워크나 라이브러리에서 사용되고 있으며, 이 API는 오직 Java에서만 제공되는, 그러니까 타언어에서는 제공되지 않는다고 한다.

## 실제 사용 사례
### JavaBean
Spring은 리플렉션을 사용해서 JavaBeans을 조작할 수 있다. JavaBean은 자동적으로 객체 생성, 의존성 주입, 어노테이션 처리 등을 수행하게 된다. 이는, 리플렉션이 필요한 객체의 메타데이터(클래스, 인터페이스, 메소드 등)를 분석 및 생성 혹은 확장, 주입을 하기 때문이다.

### Serialization, Deserialization
객체를 직렬화/역직렬화할 때, 리플렉션을 사용하여 객체의 상태를 저장하거나 복원한다. 이는, 자바의 직렬화 메커니즘이 내부적으로 리플렉션을 사용하기 때문이다.

## 어떻게 객체를 가져올 수 있을까?
결론적으로 말하자면, 리플렉션에서 가져오는 클래스는 Application ClassLoader에서 가져오게 된다. 그렇다면, 클래스 로더와 그 작동 방식을 구체적으로 학습하면서 어떻게 가져올 수 있었던 건지 알아보려고 한다.  

### ClassLoader란?
![image](https://github.com/user-attachments/assets/c4b5ea8c-0868-431d-8f49-a80c8b67d1cc)  
**JVM(Java Virtual Machine)**의 구성 요소로서, 런타임에 필요 클래스들을 로딩, 링킹, 초기화 과정을 거쳐 이를 메모리 영역인 **Runtime Data Areas**에 배치하는 역할을 수행한다. 그리고 Java Byte를 JVM으로 로드하는 **JRE(Java Runtime Environment)**의 일부이기도 하다. 클래스로더는 .class 파일을 한 번에 모두 읽어들인다. 때문에, JVM은 기본 파일이나 파일 시스템에 대해 알 필요가 없으며, 런타임의 특성을 고려해 클래스를 한 번에 불러들이지 않고, 필요할 때 불러들인다. 즉, 클래스 로더는 런타임 때, 클래스를 메모리에 적재하는 역할을 수행하는 JVM 구성요소이다.  

### 구조
![jvmclassloader](https://github.com/user-attachments/assets/3aba8972-846a-47fb-b11e-705bc663a7f2)  
출처:[https://www.geeksforgeeks.org/classloader-in-java](https://www.geeksforgeeks.org/classloader-in-java)  

클래스로더는 계층구조로 설계되어 있으며, 각 세 개의 클래스로더로 유형적으로 분류되며, 역할이 정해져 있다. 이들은 필요 시 위임을 통해 작업을 수행하게 된다.  
 
1. **Bootstrap ClassLoader(원시 ClassLoader)**  
JVM의 기본 시스템을 로드하는 인스턴스 로더로서, JVM의 일부다. 부트스트랩 클래스 로더는 Java 8 까지는 rt.jar에서 로드했으나, 9 이후부터는 JRT(Java Runtime Image)에서 로드한다. 가장 상위에 위치해 있기 때문에, 독립적으로 작동한다.  

    Object 클래스, 기본 Java API를 로드한다.   
2. **플랫폼 클래스 로더(확장 클래스 로더)**  
Java 9 이전의 Java 버전에는 확장 클래스 로더가 있었지만, 이후부터는 플랫폼 클래스 로더로 바뀌었다. JDK의 모듈 시스템에서 플랫폼별 확장 기능을 로드를 수행하거나, Java 런타임 이미지나 시스템 속성 java.platform 또는 –module-path에서 지정한 다른 모듈에서 파일을 로드한다.  

    기본 Java API를 제외한 확장 클래스들을 로드한다.  
3. **시스템 클래스 로더(애플리케이션 클래스 로더)**  
일반적으로 애플리케이션 단에서의 클래스를 불러들이기 때문에 애플리케이션 클래스 로더라고 불린다.  

    런타임에서 생성한 클래스들을 로드한다.  

### 클래스 로더 기능 원리
1. **위임 모델**  
위임 계층 알고리즘에 따라 동작하게 된다. JVM이 .class를 로드하지 않았다면, 클래스 로더는 위임 계층에 따라 체인 방식으로 로드 프로세스를 위임하게 된다. 구체적으로 동작 순서는 애플리케이션 가장 밑에서부터 시작해서 플랫폼, 부트스트랩 로더까지 Bottom-up 구조적 방식으로 동작하게 되는데, 각 위치에서 해당 클래스가 존재하지 않을 경우 이를 다음 로더에게 위임한다.  
2. **가시성 원칙**  
상위 클래스로더가 로드한 클래스는 자식 클래스로더에서 볼 수 있다. 하지만, 그 역은 볼 수 없다. 이렇게 함으로써 자식 클래스가 부모 클래스로더의 클래스를 호출하는 일이 없게 만든다. 이 원칙에 따라 캡슐화가 보장되고, 각 클래스로더에서 로드된 클래스 간의 충돌을 방지할 수 있다. 이는, 다음 특성인 고유성 때문이기도 하다.  
* **ex** -> Hyeok.class를 플랫폼 클래스로더가 로드한다면 플랫폼 클래스로더와 애플리케이션 클래스로더만 표시된다. 만약, 부트스트랩 클래스로더가 로드를 시도하면 "java.lang.ClassNotFoundException" 예외가 발생한다.  
3. **고유성 속성**  
반복 호출이 되지 않게 클래스는 한 번만 로드된다. 만약, 클래스를 찾으려고 하는데, 찾을 수 없다면 최종 root까지 찾으려고 시도를 한다. 이 과정에서 가시성 원칙이 적용된다. 

### 정리
클래스로더가 무엇이고, 구조와 특징까지 학습하였다. 이를 통해 .class를 JVM의 클래스 로더가 로드하여 런타임 때, 리플렉션이 객체를 가져오려고 명령을 내리면 Application ClassLoader가 이를 계층구조에 따라 해당 객체 클래스를 탐색하게 되고, 못 찾을 경우 자신의 부모 클래스로더로 위임 방식으로 확장적 탐색 방식을 통해 가져오게 된다는 동작 방식을 이해할 수 있게 되었다.  

## 장.단점  
**유연성, 확장성**  
객체 구조 파악, 정보 습득, 수정, 확장 등 웬만한 것들이 다 가능하다.(테스트 코드, 라이브러리) 등  
**성능 저하**  
일반적인 메소드 호출보다 처리 속도가 느려질 수 있다, 런타임에 메소드나 필드에 접근하기 위한 추가적 비용이 요구되기 때문이다.    
**내부 노출**  
비공개 멤버에 접근할 수 있기 때문에 캡슐화를 위반하며, 예상치 못한 에러가 발생할 수 있어서 보안에 취약하다.  
**가독성과 유지보수성에 좋지 않다.**  
리플렉션을 사용한 코드는 직관적이지 않고, 디버깅이 어려울 수 있다.  

## Reflection API
* **패키지**  
java.lang.reflect를 등록해야 한다.  
* **getClass()**  
java.lang.Object에서 상속받은 getClass() 메서드를 호출하면 실제 구현된 Class의 이름을 가져오는 데 사용.
* **class.forName**  
지정된 클래스 로드.
* **getConstructors()**   
객체가 속한 클래스의 공개 생성자를 가져오는 데 사용.  
* **getMethods()**   
객체가 속한 클래스의 공개 메서드를 가져오는 데 사용.  
* **getDeclaredMethod()**  
```Java
    형식 -> Class.getDeclaredMethod(name, parametertype)

    Class clazz = Class.forName("java.lang.String");
    Method[] methods = clazz.getDeclaredMethods();
    for (Method method : methods) {
        System.out.println(method.getName());
    }
```
객체에 정의된 메서드 목록 생성.  
* **invoke()**  
```Java
    Method.invoke(Object, parameter)
```
런타임에 클래스의 메서드를 호출합니다.  

**참고: parameter 기입하지 않을 경우 null 처리**  

* **Class.getDeclaredField(FieldName)**  
비공개 필드를 가져오는 데 사용, 지정된 필드 이름에 대한 Field 유형의 객체를 반환.  
* **Field.setAccessible(true)**  
필드에 사용된 액세스 수정자와 관계없이, 필드에 액세스할 수 있도록 허용.  
<br>

더 쉽고, 자세한 정보가 필요하다면 [☕ 누구나 쉽게 배우는 Reflection API 사용법](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%88%84%EA%B5%AC%EB%82%98-%EC%89%BD%EA%B2%8C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Reflection-API-%EC%82%AC%EC%9A%A9%EB%B2%95)을 참고하는 것도 좋을 것 같다!  
<br>

# 정리
지금까지 리플렉션 개념과 객체를 런타임에 어떻게 가져올 수 있는지에 대한 핵심 개념인 ClassLoader의 구조와 동작 원리를 학습하였다. 
특히 역직렬화 이슈를 해결하는 방법을 다루면서, Json 키가 생성되지 않은 Java 객체와 어떻게 매핑을 비교할 수 있었는지에 대한 의문이 있었는데, 이번 시간을 통해 그 궁금증을 해소할 수 있었다.  
<br>

# 참고
* [Using Java Reflection](https://www.oracle.com/technical-resources/articles/java/javareflection.html)
* [Introduction to Java Reflection](https://dev.java/learn/introduction_to_java_reflection/)
* [Reflection in Java](https://www.geeksforgeeks.org/reflection-in-java/)
* [ClassLoader in Java](https://www.geeksforgeeks.org/classloader-in-java/)
* [자바 리플렉션(Reflection) 이해하기](https://f-lab.kr/insight/understanding-java-reflection)
* [Java의 클래스 로더](https://www.baeldung.com/java-classloaders)
* [JVM 내부 구조 & 메모리 영역 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-JVM-%EB%82%B4%EB%B6%80-%EA%B5%AC%EC%A1%B0-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%98%81%EC%97%AD-%EC%8B%AC%ED%99%94%ED%8E%B8)
* [[조금 더 깊은 Java] JRE의 Classloader 에 대해 알아보자](https://wonit.tistory.com/590)
* [☕ 누구나 쉽게 배우는 Reflection API 사용법](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EB%88%84%EA%B5%AC%EB%82%98-%EC%89%BD%EA%B2%8C-%EB%B0%B0%EC%9A%B0%EB%8A%94-Reflection-API-%EC%82%AC%EC%9A%A9%EB%B2%95)