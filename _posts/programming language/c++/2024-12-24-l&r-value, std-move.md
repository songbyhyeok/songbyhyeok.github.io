---
title: L&R-Value, std::move
categories: cpp
---

# 개요
메모리 자원 관리, 성능 최적화를 하기 위한 기반 개념으로서 반드시 학습해두는 것이 좋으며, 다음 두 가지 선행 지식을 요구한다.   
1. call by value, reference 개념을 이해하고 있어야 한다.
2. 변수, 포인터, 특히 참조 개념을 이해하고 있어야 한다.  
  
학습을 통해 다음과 같은 이점들을 누릴 수 있다.
1. (생성자 & 연산자) 다중정의 개념, 동작 이해, 임의 조작을 할 수 있다.
2. 코드 정의 과정에서 불필요한 복사를 막아 메모리 관리 및 성능 최적화를 할 수 있다.

# 참조(Reference)
포인터와 유사한 개념으로 '&' 기호를 사용해서 가리키는 변수로서, 메모리 주소에 직접적으로 '참조'한다.  
초기화 이후 변경 할 수 없으며, 직접적인 접근이 가능하기 때문에 또 하나의 '별명'으로 불린다.

    Type & refVar = val;
  
해당 주제의 두 키워드 범주에 속하기 때문에 간단하게만 소개하고 넘어간다.  
참조는 크게 l-value, r-value 두 방식으로 구분된다.
## l-value
* 메모리 상의 주소를 가리키는 값. 즉, 저장된 변수나 객체를 뜻한다.  
* 선언 및 할당, 대입에 있어 위치는 왼쪽과 우측에 모두 나타날 수 있다.
* 참조 호출 키워드는 '&'을 사용한다.

## r-value
* 메모리 상에 할당된 주소가 없는 값으로, 주로 임시적으로 생성되는 상수(리터럴)나 연산 및 반환 값(객체)를 뜻한다.  
* 코드상에서 우측에만 사용될 수 있는 값  
* 참조 호출 키워드 '&&'을 사용한다.  

<br>

두 개념을 정리하면 다음과 같이 표현할 수 있다.  
**모든 l-value는 r-value이지만, 모든 r-value가 l-value인 것은 아니다.**  

## 코드로 구분
### case 1. 변수로 구분
```
#include <iostream>
using namespace std;

int main() {
    // a는 Lvalue, 이 위치에 값을 저장할 수 있다.
    int a = 5; 
    int b = 10;
    
    // 10 + 5는 Rvalue 계산된 값이며, 메모리 상에 위치가 없다.
    a = b + 5; // b + 5는 Rvalue
    
    return 0;
}
```

### case 2. 함수로 구분
```
#include <iostream>
using namespace std;

void foo(int& x) {
    cout << "Lvalue 참조 : " << x << endl;
}

void foo(int&& x) {
    cout << "Rvalue 참조 : " << x << endl;
}

int main() {
    int a = 5;   // Lvalue
    foo(a);      // Lvalue 참조

    foo(10);     // Rvalue 참조 (10은 임시값이므로 Rvalue)

    return 0;
}

출력:
Lvalue 참조 : 5
Rvalue 참조 : 10
```

## std::move
* l-value를 r-value로 변환하는 함수  
* 주로 자원 관리나 이동 시에 사용(ex 객체를 이동할 때 사용)

```
#include <iostream>
#include <utility>  // std::move

using namespace std;

class MyClass {
public:
    MyClass() { cout << "생성자 호출" << endl; }
    MyClass(const MyClass& other) { cout << "복사 생성자 호출" << endl; }
    MyClass(MyClass&& other) { cout << "이동 생성자 호출" << endl; }

    ~MyClass() { cout << "소멸자 호출" << endl; }
};

int main() {
    MyClass obj1;                  // 생성자 호출
    MyClass obj2 = std::move(obj1); // 이동 생성자 호출 (std::move가 Lvalue를 Rvalue로 변환)

    return 0;
}

출력:
생성자 호출
이동 생성자 호출
소멸자 호출
```

# 참고
* [참조자](https://www.tcpschool.com/cpp/cpp_cppFunction_reference)
* [C++에서 임시 객체와 l-value, r-value의 이해](https://f-lab.kr/insight/understanding-temporary-objects-and-lvalue-rvalue-in-cpp-20240608)