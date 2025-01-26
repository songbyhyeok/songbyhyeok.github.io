---
title: Extern keyword
categories: c
---

# Extern?
**다른 소스 파일의 전역 변수를 현재 사용하는 공간에 가져오고 싶을 때 사용하는 키워드로서, global 변수로 불린다.**  

## 코드로 이해
### main.c, print.c
![1](https://github.com/user-attachments/assets/26fdf7ba-ba87-46cb-9b5e-51b1b4009af5)  
두 개의 소스 파일 'main.c', 'print.c'  

### print.c
![2](https://github.com/user-attachments/assets/2a764d9a-737b-4279-8c2e-9fa1410091e6)  

### main.c
![3](https://github.com/user-attachments/assets/58bfdab6-dada-47c4-93a0-6482cb06f694)

'print.c'의 전역변수 'num222'를 'main.c'로 가져와 사용하고 싶을 때, **'extern keyword'** 를 'main.c'의 전역란에 extern과 num222 전역변수명을 함께 명시하여 외부에 있다는 것을 알리면 사용할 수 있다.

## 생성 과정에서 이해
두 개의 소스 파일들이 컴파일러에 의해 기계어가 포함된 목적파일(.o)로 변환 후, 링킹 과정에서 링커에 두 개를 묶어 global 변수로서 사용이 가능하게 된 것.  
<br>

# 참고
* [C++ 전역변수의 static 과 extern 키워드](https://mr-dingo.github.io/c/c++%EB%BD%80%EA%B0%9C%EA%B8%B0/2019/01/10/static&extern.html)