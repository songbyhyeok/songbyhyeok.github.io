---
title: 프로그래밍 패러다임
categories: lang_theory
---

# 패러다임
패러다임(영어: paradigm)은 어떤 한 시대 사람들의 견해나 사고를 근본적으로 규정하고 있는 테두리로서의 인식의 체계, 또는 사물에 대한 이론적인 틀이나 체계를 의미하는 개념이다. - 위키백과 -   

**프로그래밍 패러다임이란?**   
트렌드에 맞춰, 특정 프로그래밍 언어를 기반으로 주어진 도구와 기술을 활용하는 문제 접근 및 해결하는 방법론을 말한다.  
패러다임은 한 언어에 종속되지 않으며, 요구조건에 따라 여러 전략으로 나누어진다.   
<br>

# 명령형 프로그래밍 패러다임(Imperative programming paradigm)
폰 노이만(Von Neumann) 설계를 기반으로, 역사와 함께한 프로그래밍 패러다임 중 하나이며, 해당 방식은 **어떻게 문제를 해결할 것인가**에 초점을 두고 있다. 이 패러다임은 코드의 상태와 상태를 변경시키는 구문 형식으로, 수행 명령어 단위를 순차적으로 지시 및 처리한다.  

```Java
public class Main {
    public static void main(String[] args) {
        // Array to store marks
        int[] marks = {12, 32, 45, 13, 19};

        // Variable to store the sum of marks
        int sum = 0;

        // Variable to store the average
        float average = 0.0f;

        // Calculate the sum of marks
        for (int i = 0; i < 5; i++) {
            sum = sum + marks[i];
        }

        // Calculate the average
        average = sum / 5.0f;

        // Output the average
        System.out.println("Average of five numbers: " + average);
    }
}
```
## 절차적 프로그래밍 패러다임(Procedure)
기계 모델에 기반한 프로그래밍 패러다임이다. 명령형 패러다임과 큰 차이가 없으며, 코드 재사용이 가능해서 과거에 큰 장점이 되었다.
```Java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        // Create a Scanner object for reading input
        Scanner scanner = new Scanner(System.in);

        // Prompt user to enter a number
        System.out.println("Enter any Number: ");

        // Read number from user input
        int num = scanner.nextInt();

        // Initialize factorial to 1
        int fact = 1;

        // Calculate factorial using a for loop
        for (int i = 1; i <= num; i++) {
            fact = fact * i;
        }

        // Print the factorial of the number
        System.out.println("Factorial of " + num + " is: " + fact);
    }
}
```

## 개체 지향 프로그래밍 (Object-Oriented) 
데이터가 "객체" 내에 캡슐화되고 구성 요소 부분이 아닌, 객체 자체가 운용되는 프로그래밍 접근 방식 - mdn web docs -  

프로그램이 클래스와 객체의 집합으로 구성되고, 상태와 행동을 하는 즉, 행위를 하는 객체들 간의 상호작용을 통해 구현되는 방법론이다.
```Java
import java.io.*;

class GFG {
    public static void main(String[] args)
    {
        System.out.println("GfG!");
        Signup s1 = new Signup();
        s1.create(22, "riya", "riya2@gmail.com", 'F',
                  89002);
    }
}

class Signup {
    int userid;
    String name;
    String emailid;
    char sex;
    long mob;

    public void create(int userid, String name,
                       String emailid, char sex, long mob)
    {
        System.out.println(
            "Welcome to GeeksforGeeks\nLets create your account\n");
        this.userid = 132;
        this.name = "Radha";
        this.emailid = "radha.89@gmail.com";
        this.sex = 'F';
        this.mob = 900558981;
        System.out.println("your account has been created");
    }
}
```

## 병렬 처리 접근법
프로그램 명령어를 여러 프로세서에 나누어 동시에 수행.처리하기 때문에, 짧은 시간 안에 효율적으로 처리할 수 있다. 해당 기법은 분할 정복 방식과 유사하다.  
<br>
<br>

# 선언형 프로그래밍 패러다임(Declarative programming paradigm)
선언형 프로그래밍은 제어 흐름에 대해 언급하지 않고 계산의 논리를 표현하는 방식으로 프로그램을 작성하는 스타일이다. 이 패러다임의 핵심은 "어떻게 할 것인지"보다는 "**무엇을 할 것인지**"에 초점을 맞추며, 코드가 실제로 무엇을 하는지에 중점을 둔다.

## 함수형 프로그래밍 패러다임
함수형 프로그래밍 패러다임은 수학에 뿌리를 두고 있으며, 특정 프로그래밍 언어에 구애받지 않는다. 핵심 원칙은 일련의 수학적 함수들을 실행하는 것이다. 추상화의 중심은 특정 계산을 수행하는 함수이며, 데이터 구조는 그다지 중요한 요소가 아니다.
```
Examples of Functional programming paradigm:
JavaScript : developed by Brendan Eich
Haskell : developed by Lennart Augustsson, Dave Barton
Scala : developed by Martin Odersky
Erlang : developed by Joe Armstrong, Robert Virding
Lisp : developed by John Mccarthy
ML : developed by Robin Milner
Clojure : developed by Rich Hickey 
```

## 데이터 기반 프로그래밍 접근법
프로그램 명령문이 수행할 단계의 순서를 정의하는 대신 일치시킬 데이터와 필요한 처리를 설명하는 프로그래밍 패러다임이다. 이 접근 방식은 알고리즘이나 제어 구조는 덜 강조하면서 프로그램 설계에 영향을 미치는 주요 요소로 데이터를 우선시한다. 데이터 기반 프로그래밍은 끊임없이 변화하고 다양하며 복잡한 대량의 데이터를 처리하는 애플리케이션에 유연성, 확장성 및 유지 관리 측면에서 상당한 이점을 제공할 수 있다.
```
CREATE DATABASE databaseAddress;
CREATE TABLE Addr (
    PersonID int,
    LastName varchar(200),
    FirstName varchar(200),
    Address varchar(200),
    City varchar(200),
    State varchar(200)
); 
```
<br>

# 참고
- [Introduction of Programming Paradigms](https://www.geeksforgeeks.org/introduction-of-programming-paradigms/)
- [2. Programming Paradigm](https://www.youtube.com/watch?v=TLlndTSEWWg&ab_channel=%EA%B3%B0%ED%8A%80%EA%B9%80)
- [프론트엔드 개발에서의 명령형과 선언형 프로그래밍 비교](https://f-lab.kr/insight/imperative-vs-declarative-programming-in-frontend-development)
- [패러다임](https://ko.wikipedia.org/wiki/%ED%8C%A8%EB%9F%AC%EB%8B%A4%EC%9E%84)
- [데이터 기반 프로그래밍](https://appmaster.io/ko/glossary/deiteo-giban-peurogeuraeming)
- [논리 프로그래밍](https://appmaster.io/ko/glossary/nonri-peurogeuraeming)