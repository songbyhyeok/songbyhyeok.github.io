---
title: L-Value vs R-Value, std::move
categories: cpp
---

# 개요
# 참조(Reference)
포인터와 유사한 개념으로 '&' 기호를 사용해서 가리키는 변수로서, 메모리 주소에 직접적으로 '참조'한다. 초기화 이후 변경 할 수 없으며, 직접적인 접근이 가능하기 때문에 또 하나의 '별명'으로 불린다. 
  
해당 주제의 두 키워드 범주에 속하기 때문에 간단하게만 소개하고 넘어간다.  
참조는 크게 l-value, r-value 두 방식으로 구분된다.
## l-value
'&' 사용
```
```

## r-value
'&&' 사용
객체의 값을 다른 객체로 이동하는 용도로 해당 참조가 유용하게 쓰인다.
```
```

코드상에서 우측에만 사용될 수 있는 값이다. 

## 차이점
모든 l-value는 r-value이지만 모든 r-value가 l-value인 것은 아닙니다.

# STD::MOVE

# 정리

# 참고
* [참조자](https://www.tcpschool.com/cpp/cpp_cppFunction_reference)