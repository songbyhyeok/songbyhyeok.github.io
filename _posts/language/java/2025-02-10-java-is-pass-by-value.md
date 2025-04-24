---
title: Java는 pass by value 라고 ??
categories: java
---

# 자바는 객체 주소를 전달하는 방식이다.
“Java는 Call by value = Pass by value만 존재한다”란 글을 읽고 여러 가지 면에서 혼란스러웠는데, 이것을 이해/정리 목적 취지로 글을 쓰게 되었다.  

## 자바는 pass by ref가 아니야 ?
### 상태 변경만 가능하다.
```Java
public class Main {
     public static void main(String[] args) {
          Foo f = new Foo("f");
          changeReference(f); // It won't change the reference!
          modifyReference(f); // It will modify the object that the reference variable "f" refers to!
     }

     public static void changeReference(Foo a) {
          Foo b = new Foo("b");
          a = b;
     }

     public static void modifyReference(Foo c) {
          c.setAttribute("c");
     }
}
```

![Image](https://github.com/user-attachments/assets/cb21e1b9-7be4-41b6-8345-7b6f69a05cc7)  
![Image](https://github.com/user-attachments/assets/53209120-0ec6-4520-84a1-3c20180314b9)  
![Image](https://github.com/user-attachments/assets/10dd2b21-c8fc-4313-a9fa-5a051dd29b22)  

### “객체” 값에 의한 전달
```Java
1. Person person;
2. person = new Person("Tom");
3. changeName(person);
4.
5. //I didn't use Person person below as an argument to be nice
6. static void changeName(Person anotherReferenceToTheSamePersonObject) {
7.     anotherReferenceToTheSamePersonObject.setName("Jerry");
8. }
```

![Image](https://github.com/user-attachments/assets/8e033e4f-2927-4d39-9fc5-514bdb241ce8)  

person 객체 주소 값이 3bad086a라면, 매개변수 anotherReferenceToTheSamePersonObject 도 3bad086a 주소 값을 가지게 되어, 두 변수는 공통의 힙 주소 값을 가리키게 된다. 즉, 참조에 의한 전달이 아니라 참조 값에 의한 전달이 되는 셈이다.  

## pass by ref에 대해서
### call by vs pass by?
call by ref는 많이 들어봤는데?? 그렇다면, pass by는 무엇일까?? 차이점을 한 번 찾아보았다.  

![Image](https://github.com/user-attachments/assets/b5014388-12c1-4b38-b80c-aaa29d33cf1d)  
![Image](https://github.com/user-attachments/assets/405a70a3-f715-4d66-be6c-7661fd3ab4ae)  

~에 의한 호출과 ~에 의한 전달은 같은 같은 개념으로 보는 것이 맞다. 

### c++의 전달 방식
c++에서 함수 호출 방식은 세 가지가 있다.  

**1. Call by Value → 값에 의한 호출**
```Cpp
void foo(int val) { // 값에 의한 호출/전달
	
}

int main() {
	int val = 0;
	foo(val); // 값을 복사한다.

	return 0;
}
```

**2. Call by Address → 주소에 의한 호출, 포인터를 이용한 전달 방식**
```Cpp
void foo(Object * obPtr) { // 포인터로 전달 받는다.
	
}

int main() {
	Object ob = new Object();
	foo(&ob); // 주소를 전달한다.

	return 0;
}
```
- 함수 호출 시 변수의 주소를 인자로 넣어서 호출하는 방식
- 포인터 변수는 다른 변수의 주소로 바꿀 수 있다.  
<br>

**3. Call by Reference → 참조에 의한 호출, 참조자**
```Cpp
void foo(int & refOb) { // 참조 매개변수, refOb는 ob의 객체 주소 값을 가진 또 다른 변수
	
}

int main() {
	Object ob = new Object();
	foo(a); // Call by Value 처럼, 객체 주소 값에 의한 전달

	return 0;
}
```
- 참조자는 대상에게 또 다른 이름을 붙여주는 별칭이다.
- 매개변수는 인자 변수의 객체 주소 값으로 초기화가 된다.
- 한 번 초기화가 되면, 참조 대상을 바꿀 수 없다.  

### 여전히 pass by value 일까?
자바는 “객체 주소 값에 의한 전달”인 적절한 표현이 없어 call by value로 정의를 내렸다. 그러나, C++에서 함수 호출 방식을 학습하면서 call by reference는 객체 주소 값을 공유하는 것을 알 수 있게 되었다.  

따라서 나는 pass by value 보다는 call by reference 란 표현이 적절하다고 생각한다.  

## 정리
java는 주장대로 pass by value가 맞다(값이 객체 주소에 의한 전달이라면). 그렇지만 객체 전달/호출에 의한 주소 값을 공유한다는 건 엄밀히 말해서 pass by reference다. 그 이유는 하드웨어와 근접한 c++에서 전달 방식 중에 call by addess와 call by reference가 있고, 그리고 java가 객체에 대해 취하는 동작은 명확하게 c++의 call by reference이기 때문이다.  

따라서 포인터처럼 객체를 핸들링하고 싶다는 이유라고 한다면, java is pass by value가 아니라, java is not call by address가 맞는 표현이 아닐까 싶다.  

## 참고
- ### [Java에는 Call-By-Value(Pass-By-Value)만 존재한다.](https://yangbox.tistory.com/51)
- ### [Is Java "pass-by-reference" or "pass-by-value"?](https://stackoverflow.com/questions/40480/is-java-pass-by-reference-or-pass-by-value)