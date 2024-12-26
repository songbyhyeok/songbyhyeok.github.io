---
title: Register keyword
categories: c
---

# 개요
C언어는 값을 CPU 레지스터에 저장할 수 있는 register 키워드를 제공한다.  
이 키워드를 사용하여 레지스터 변수의 특성과 일반 변수와의 차이점에 다뤄보려고 한다.  

## Register?
**계층구조**  
![image](https://github.com/user-attachments/assets/67015403-a590-4063-b21b-08c55feb957a)  

주기억장치에서 읽어온 명령어와 데이터를 효율적으로 처리하는 임시 저장 장치로, 가장 빠른 접근 속도와 처리 성능을 자랑하지만, 개수 제한과 높은 비용, 그리고 용량이 제한적이라는 단점이 있다.  

### 레지스터 변수를 제공하는 이유
C는 고급언어지만, 하드웨어에 가까운 저수준 언어의 특성도 가지고 있어, 시스템 프로그래밍에 적합하다. 따라서 레지스터 변수로 값을 직접적으로 저장하고 제어하여 컴파일 최적화를 할 수 있다. 그러나 다음과 같은 이유로 사용하지 않게 되었다.   
1. **레지스터 수의 제한**  
레지스터는 수가 제한적이므로, 'register' 키워드를 사용한다고 해서 모든 변수가 반드시 레지스터에 저장되는 것은 아니다.  
2. **하드웨어, 컴파일러의 성능 발전**  
과거에는 하드웨어 성능이 부족하여 최적화가 중요했지만, 현재는 컴파일러의 성능이 발전하여 자동으로 최적화를 수행하므로, 더 이상 개발자가 직접 최적화에 신경 쓸 필요가 없다.   

### 코드 관점
```
// 코드 출처 코딩 도장
#include <stdio.h>
#include <stdlib.h>    // malloc, free 함수가 선언된 헤더 파일

int main() {
    register int num1 = 10;      // 변수 num1은 CPU의 레지스터를 사용

    printf("%d\n", num1);
    // printf("%p\n", &num1);    // 컴파일 에러. num1은 메모리에 없으므로 메모리 주소를 구할 수 없음
                                 // error C2103: 레지스터 변수에 '&'이(가) 있습니다.

    register int *numPtr = malloc(sizeof(int));

    // 레지스터 변수에 메모리 주소는 저장할 수 있으므로 역참조 연산자를 사용할 수 있음
    *numPtr = 20;
    printf("%d\n", *numPtr);     // 20

    free(numPtr);

    return 0;
}
```

**속도 비교**  
```
// 코드 출처 [C/C++] int와 register int의 속도 차이
    int main() {    
	int tmp1 = 0;    
	int tmp2 = 0;    
	clock_t clk1 = clock();    
	for (int i = 0; i < 10000; i++) {        
		for (int j = 0; j < 10000; j++) {            
			tmp1 += i;        
		}    
	}    
	cout << "1: " << clock() - clk1 << "ms" << endl;        
	
	clock_t clk2 = clock();    
	for (register int i = 0; i < 10000; i++) {        
		for (register int j = 0; j < 10000; j++) {            
			tmp2 += i;        
		}    
	}    
	cout << "2: " << clock() - clk2 << "ms" << endl;
}

```

위 코드에서 스택에 저장된 변수들(즉, 1~2 라인의 변수)의 속도는 250ms로 측정되었으나, 레지스터 변수를 선언한 두 번째 반복문에서는 236ms로 약간 더 빠른 결과를 보였다. 그러나 레지스터 변수를 사용하는 것이 항상 좋은 선택이라고 할 수는 없다. 레지스터 수는 한정적이며, 컴파일러가 최적화 과정을 자동으로 처리하기 때문이다.  

# 참고
* [79.4 레지스터 변수 사용하기](https://dojang.io/mod/page/view.php?id=804)  
* [[C/C++] int와 register int의 속도 차이](https://blog.readiz.com/281)