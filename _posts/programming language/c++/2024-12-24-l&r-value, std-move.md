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
<br>

# 참조(Reference) 분류
포인터와 유사한 개념으로 '&' 기호를 사용해서 가리키는 변수로서, 메모리 주소에 직접적으로 '참조'한다.  
초기화 이후 변경 할 수 없으며, 직접적인 접근이 가능하기 때문에 또 하나의 '별명'으로 불린다.

    Type & refVar = val;
  
해당 주제의 두 키워드 범주에 속하기 때문에 간단하게만 소개하고 넘어간다.  

## values
<img src="https://github.com/user-attachments/assets/438ce50b-d063-45d2-a846-186fd8b3b80e" style="width: 50%; height: auto;">  
출처: https://learn.microsoft.com/ko-kr/cpp/cpp/lvalues-and-rvalues-visual-cpp?view=msvc-170&viewFallbackFrom=vs-2019  
<br>
참조는 크게 gl-value, r-value 두 방식으로 구분되며, 더 세분화가 가능하다. (위 다이어그램을 참고)  

### l-value
* gl-value 범주에 포함
* 메모리 상의 주소를 가리키는 값. 즉, 저장된 변수나 객체를 뜻한다.  
* 위치는 왼쪽과 우측에 모두 나타날 수 있다. 
* 참조 호출 키워드는 '&'을 사용한다.
* 복사 (생성, 대입) 연산자에 사용된다.  

### r-value
* 특정 값이나 표현식의 결과로, 임시적인 값을 뜻한다. 메모리의 특정 주소를 가지지 않으며, 대개 변수나 객체의 값 자체(rodata)
* 코드상에서 우측에만 사용될 수 있는 값  
* 참조 호출 키워드 '&&'을 사용한다.  
* 이동 (생성, 대입) 연산자에 사용된다.

**.rodata?**  
읽기 전용 전역 상수들을 저장하는 공간으로서, os 메모리 구조의 전역변수를 저장하는 데이터 영역의 구성 중 하나다.

## 코드로 이해하기.
### case 1. 변수로 구분
```
#include <iostream>
using namespace std;

int main() {
    // a는 Lvalue, 이 위치에 값을 저장할 수 있다.
    int a = 5; 
    int b = 10;
    
    // 10 + 5는 Rvalue 계산된 값이며, rodata 영역
    a = b + 5; // b + 5는 Rvalue
    
    return 0;
}
```

### case 2. 함수로 구분
```
#include <iostream>
using namespace std;

/* 1. call by ref를 생각하면 된다. lvalue를 참조하는 방식으로, 변수나 객체의 "주소"를 전달하는 방식이다.  
함수 내부에서 참조를 통해 같은 주소 값을 공유하게 된다. */
/* 2. [복사]생성자, 대입 연산자 구현에 사용 된다. 전달된 lvalue는 앞에 반드시 const를 붙여 안정성을 높인다. */
void foo(int& x) {
    cout << "Lvalue 참조 : " << x << endl;
}

/* 1. rvalue를 참조하는 방식이다. 임시 객체나 리소스의 소유권을 이동시킬 때 사용된다.*/
/* 2. rvalue는 더 이상 사용되지 않는 값으로, 이를 참조하는 방법은 리소스 소유권을 이동시키거나 최적화에 유용하다. */
/* 3. [이동]생성자, 대입 연산자 구현하는 데 사용된다.
void foo(int&& x) {
    cout << "Rvalue 참조 : " << x << endl;
}

int main() {
    int a = 5;   // Lvalue
    foo(a);      // Lvalue 참조

    foo(10);     // Rvalue 참조 (10은 임시값이므로 Rvalue)
    //foo(move(a)) // lvalue를 rvalue로 변환시켜 참조

    return 0;
}

출력:
Lvalue 참조 : 5
Rvalue 참조 : 10
```

매개변수에 전달되는 참조 표현식의 종류에 따라 함수 오버로딩이 결정된다.

## 참조 전달 방식에 따른 메모리 할당 과정
```
#include <iostream>

using namespace std;

void storeByV(string s) {
	string s2 = s;
}

void storeByLv(string& s) {
	string s2 = s;
}

void storeByRv(string&& s) {
	string s2 = s;
}

int main() {
	string str = "hello world";
	
	storeByV(move(i));
	storeByLv(str);
	storeByRv(move(i));
	
	cout << str << ' ';
0
	return 0;
}
```

**참조 표현식에 따라 메모리 영역의 스택, 힙에서 어떻게 할당 및 처리가 되는 건지 알아보자.**

### pass by value
![image](https://github.com/user-attachments/assets/94a93141-2982-4bba-b558-c65c34a568bd)  
storeByV를 호출하면 매개변수가 기본자료형이기 때문에 값 복사가 일어나게 되는데, main 지역 변수 str은 heap에 있는 값을 가리키고 있으므로, 매개변수 s는 힙의 새로 할당된 동일한 값을 가리키게 되는 것이다. 내부 함수에는 임시객체인 지역변수 s2가 존재한다. 그리고 s는 s2에게 대입연산자를 사용했기 때문에 이것도 마찬가지로 힙에 새로 값을 할당 및 이를 s2가 가리키게 되어 총 2번의 복사가 일어나게 된다.

### l-value 참조
![image](https://github.com/user-attachments/assets/a29bcab3-82b9-4237-87c3-cabea974cc1a)  
pass by ref에 의해 매개변수 s는 동일 주소를 가리키게 된다. 즉 변수 저장 공간 자체는 다르지만, 힙의 동일 주소를 가리키고 있게 되는 것이다. 따라서 copy가 발생되지 않는다. (엄밀히 말해 지역변수 copy는 발생했다. 하지만 기본타입의 크기는 매우 작아서 비용 처리를 하지 않는다.) 이후에 내부함수 s2가 생성 및 s에 의해 복사가 일어나서 1 copy가 발생된다.  

### r-value 참조
![image](https://github.com/user-attachments/assets/5e1496fa-eeeb-4dfc-b418-a7305092985d)  
전달된 인자는 힙 주소 값 소유권을 매개변수 s에게 이전하였다. 따라서 0 copy면서, s는 동일 힙 주소 값을 가리키게 된다. 이때, 매개변수 s는 r-value 상태이고, 내부 함수에서 s2가 생성 및 s가 복사 대입을 할 때, 이때는 s는 l-value로 다시 형변환이 일어나게 되고 1 copy가 발생한다.

## 간단 명료한 l-value와 r-value 차이점
**-MS 공식문서-**  
모든 l-value는 r-value이지만, 모든 r-value가 l-value인 것은 아니다.  
```
int iv = 0;
iv = 5 + iv; <- r-value 위치에 l-value가 위치함
```

l-value는 메모리 위치를 가리키므로, 값으로 사용할 수 있다. iv는 l-value이고, iv를 우변에 위치시켜 값으로 사용할 수 있기 때문에 l-value는 r-value가 되는 것이다. 하지만 r-value는 읽기 전용 상수인 즉, 값 자체이기 때문에 좌측에 위치시켜 값을 저장할 수 있지 않아, l-value가 될 수 없다.
<br>

# std::move keyword
특정 객체 t(이동 시키고 싶은 객체)가 가진 자원 소유권을 다른 객체에게 이전할 수 있는 권한을 제공하는 것.

## 왜? 실행이 아닌 권한일까?
std::move 는 객체 t를 r-value로 type casting 하기 때문이다. 좀 더 엄밀하게 표현하자면, t를 "이동 가능"한 **x-value**로 만든다.
```
// 객체를 이동시키지 않지만, 이 표현식이 "이동 가능한 상태"임을 알리는 것.
    template< class T >
    typename std::remove_reference<T>::type&& move( T&& t ) noexcept;
```

## x-value
std::move 키워드 사용 시 사용되는 용어인데, 다른 객체로 이동하거나 할당할 수 있는 값을 의미한다.  
즉, gl&r-value의 특징들을 가지고 있는 값이다.

### 이동 대입 연산자
x-value로 타입이 변경됐기 때문에, 이전 명령이 실행될 경우 이동 연산자가 호출된다.  
이는, 리소스를 다른 객체로 이동시키기 위한 하나의 함수다.

### l-value, r-value, x-value의 차이점
* l-value: 메모리 위치를 가리키며, 할당할 수 있는 위치를 제공하는 값 (예: 변수).  
* r-value: 일시적이거나 임시적인 값을 의미하며, 메모리 위치를 가리키지 않음 (예: 5, x + y).  
* x-value: 이동할 수 있는 값을 나타내며, 자원을 다른 객체로 이동시킬 수 있는 값 (예: std::move(x)).  

## std::move는 소유권을 이전한다.
```
#include <iostream>
#include <string>
#include <utility>  // std::move

using namespace std;

int main() {
    string str1 = "Hello, world!";  // str1에 값 할당

    cout << "Before move:" << endl;
    cout << "str1: " << str1 << endl;  // str1의 값 출력

    // std::move를 사용하여 소유권을 이동
    string str2 = std::move(str1);  // str1의 소유권을 str2로 이동

    cout << "\nAfter move:" << endl;
    cout << "str1: " << str1 << endl;  // str1은 이제 유효하지만 값은 예측할 수 없음
    cout << "str2: " << str2 << endl;  // str2는 str1의 값을 소유

    return 0;
}

출력:
Before move:
str1: Hello, world!

After move:
str1: 
str2: Hello, world!
```

l-value 표현식을 캐스팅하여 r-value로 이전 가능한 상태로 만들어 다른 객체로 이동될 경우, 이전 l-value였던 변수에는 더이상 아무 값도 들어가 있지 않게 된다.

## 기본타입은 이동시키지 않는다.
```
#include <iostream>

int main() {
    int iValue = 42;

    // std::move는 객체를 rvalue로 캐스팅하는 역할
    int iMoved = std::move(iValue);

    std::cout << "iValue: " << iValue << std::endl;  // iValue는 여전히 42
    std::cout << "iMoved: " << iMoved << std::endl;  // iMoved는 42
}
```

std::move의 핵심 목적은 힙에 저장된 메모리 자원의 소유권을 이동시키는 권한을 제공하는 것이다.  
예를 들어, std::vector, std::string 같은 동적 메모리를 관리하는 객체에서 std::move를 사용하면 효과를 볼 수 있다.

# 결국엔 매개변수는 pass by value 방식이 옳다.
메모리 할당 과정에서 **가장 적은 copy**가 일어나는 최적화 매개변수 정의가 왜 pass by value 인지 설명하려고 한다.

## pass by ref 방식은 반드시 const를 붙여야 한다.
해당 방식으로 전달 후, 내부에서 move 키워드로 다시 값 이동을 시킬 경우에 0 copy가 되기 때문에 이상적인 결과라고 할 수 있다. 하지만, l-value 표현식은 참조다. 그렇기 때문에 매개변수 값을 변경 시 전달자 l-value도 같이 변경되기 때문에 const를 붙여 보호해야 하는데, 문제는 상수화 처리로 인해 1 copy가 발생하게 된다. 그리고 r-value 전달을 할 수 없는 오버로딩이다. 

## pass by r-value
해당 표현식을 전달하면 해당 표현식은 삭제가 되고, 매개변수의 r-value가 전달된 힙 주소를 가리키게 되어 0 copy가 된다. (이때, c++ 컴파일러가 최적화 기법인 copy elision(복사 생략)을 사용해서 0 copy를 만들게 된다.) 이후, 내부 함수 임시 지역변수에 다시 move 키워드를 사용해 전달하면 0 copy 가 되어, 이상적인 방식이라고 할 수 있다. 하지만, 이 역시도 오직 r-value 형만 제한적으로 오버로딩 호출되는 방식이다.  

### Copy Elision(복사 생략)?
컴파일러가 최적화를 위해 복사 및 이동 (C++11 이후) 생성자를 생략하여, 복사 없는 값을 전달한다.  
크게 두 가지 경우가 있는데 Return Value Optimization(반환값 최적화)인 경우와 class type의 template object(임시 개체)가 동일한 유형의 복사될 때 이다.
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

## pass by value
해당 방식에서는 기본형 타입과 r-value 만 고려하면 된다. l-value를 제외한 이유는 const로 매개변수를 상수화 처리함으로써 결국 1 copy를 만들어지게 되는데, 기본형 타입 역시 전달할 때 1 copy는 필연적이라서 결국엔 비용이 같아진다. 때문에, move 키워드를 내부에 사용해서 전달만 하면 결국 1 copy로 끝나게 된다.  
그렇다면 r-value 전달을 하면 어떻게 될까? r-value 역시 전달 될 때 Copy Elision에 의해 0 copy가 되고 이후 move 키워드를 사용해 다시 소유권 이전을 시키기 때문에 copy가 절대 발생하지 않게 된다. 따라서 **pass by value 전달 방식을 사용하는 것이 좋다.**

### pass by value 방식에 r-value 표현식이 허용되는 이유
pass by value 방식은 전달된 인자 값의 복사본을 만들어 매개변수에 전달하게 된다. r-value는 복사 가능한 값이기 때문에, 함수 내에서 매개변수는 r-value의 복사본을 받으며, 그 복사본은 함수 종료 시 정상적으로 소멸되고 이 과정에서 소유권 이동이 일어나지 때문에 가능하다. 해당 과정은 Copy Elision에 따라 처리된다.  
<br>

# 참고
* [참조자](https://www.tcpschool.com/cpp/cpp_cppFunction_reference)
* [C++에서 임시 객체와 l-value, r-value의 이해](https://f-lab.kr/insight/understanding-temporary-objects-and-lvalue-rvalue-in-cpp-20240608)
* [C++ 레퍼런스 - std::move 함수](https://modoocode.com/301)
* [What is Values(lvalue, rvalue, xvalue, prvalue, glvalue) ?](https://dydtjr1128.github.io/cpp/2019/06/10/Cpp-values.html)
* [std::move](https://en.cppreference.com/w/cpp/utility/move)
* [BSS vs 데이터 vs rodata (전역영역 이름 구분하기)](https://juntheworld.tistory.com/118)
* [L VALUE R VALUE 레퍼런스 알아보기](https://www.youtube.com/watch?v=6buEm6R980o&list=PLDV-cCQnUlIa5K5UYxaXsH78Ao0pltrlq&index=4&ab_channel=%EC%BD%94%EB%93%9C%EC%97%86%EB%8A%94%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
* [std::move (무브 키워드) , modern C++](https://www.youtube.com/watch?v=GutCygNRi-I&list=PLDV-cCQnUlIa5K5UYxaXsH78Ao0pltrlq&index=5&ab_channel=%EC%BD%94%EB%93%9C%EC%97%86%EB%8A%94%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
* [Copy elision](https://en.cppreference.com/w/cpp/language/copy_elision)
* [C++ Copy Elision(복사 생략)](https://hodongman.github.io/2020/11/02/C++-copy-Elision-copy.html)
* [L VALUE R VALUE 레퍼런스 알아보기](https://www.youtube.com/watch?v=GutCygNRi-I&list=PLDV-cCQnUlIa5K5UYxaXsH78Ao0pltrlq&index=3&ab_channel=%EC%BD%94%EB%93%9C%EC%97%86%EB%8A%94%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)
* [std::move (무브 키워드) , modern C++](https://www.youtube.com/watch?v=6buEm6R980o&list=PLDV-cCQnUlIa5K5UYxaXsH78Ao0pltrlq&index=4&ab_channel=%EC%BD%94%EB%93%9C%EC%97%86%EB%8A%94%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D)