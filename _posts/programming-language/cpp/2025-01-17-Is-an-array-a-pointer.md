---
title: 배열은 포인터인가?
categories: cpp
---

# Issue
코딩을 하던 중 배열을 매개변수로 사용하는 상황이 생겼다. 배열의 크기를 내부 함수에서 계산하면 된다고 생각하고, 크기를 확인한 후 ‘?’ 떠올랐다. 예상했던 대로라면 10000000 바이트 크기가 나와야 했는데, 결과는 ‘8’이라는 값이 나왔다. 왜 이런 결과가 나온 것일까?
```Cpp
// 문제의 코드
bool hasPrimes[10000000];

void sieveOfEratosthenes(bool hasPrimes[]) {
    const int hPValSize = sizeof(hasPrimes);
    ...................
}

// 해결 방법
void sieveOfEratosthenes(bool hasPrimes[], int size) {.....}

int main(void) {
    sieveOfEratosthenes(hasPrimes, sizeof(hasPrimes));
}
```
## 8은 포인터 크기
![Image](https://github.com/user-attachments/assets/09109a04-90b7-4937-9004-d803736a2448)
![Image](https://github.com/user-attachments/assets/332db0a1-dc8c-4a5d-922d-0abeb20a7312)  

이때, 배열을 매개변수로 사용할 경우 배열이 실제로는 포인터로 인식된다는 점을 다시 떠올리게 되었다. 그래서 나온 값은 배열의 크기가 아니라 포인터의 크기였다. 내가 사용한 vscode IDE에서 빌드 설정을 x64로 해놨기 때문에, 포인터의 크기가 8바이트로 설정된 것이다. 이 문제는 이전부터 가끔 신경 쓰이던 부분이었는데, 이번 기회에 포인터와 배열의 차이점을 정리해 볼 필요가 있다고 생각했다.  
<br>

# 배열은 포인터일까?
배열 자체는 본질적으로 고정된 크기의 연속된 메모리 공간을 나타내지만, 포인터는 메모리 주소를 저장하는 변수이다. 따라서 목적과 역할 자체가 아에 다르다고 할 수 있다. 하지만 배열의 이름은 해당 배열의 첫 번째 요소의 주소를 가리키는 포인터처럼 동작하는 특성이 있다. 그렇기 때문에 배열은 상황에 따라 포인터처럼 동작할 수도 있고, 그렇지 않을 수도 있다. 

## 배열을 매개변수로 사용할 땐, 포인터처럼 동작한다.
![Image](https://github.com/user-attachments/assets/987eb65b-1241-4f99-b66e-0899782f2c57)    

컴파일러는 매개변수 파라미터를 포인터로 동일 취급한다.
1. **typeid()** 
런타임에 타입을 결정할 수 있는 메소드다. "int * ptr64" 라고 출력되었다.  
2. **포인터 산술 연산 사용**  
배열은 산술 연산을 사용할 수 없지만, 포인터는 산술 연산을 사용할 수 있다.  
3. **sizeof()**  
배열의 크기 5가 아닌, 포인터 크기인 8(x64)이 출력되고 있다. 

## 배열과 포인터의 차이점
포인터로도 작용하는 배열. 포인터와 구체적 차이점을 통해 구별할 필요성을 느꼈다.  

### sizeof() 크기
배열은 선언 시 크기가 정해지지만, 크기를 변경할 수 없다. 반면, 포인터는 동적으로 메모리 주소를 참조할 수 있어 크기를 변경할 수 있다.  예를 들어, 배열의 크기는 컴파일 타임에 결정되며, 포인터는 런타임에 동적으로 메모리를 할당하거나 변경할 수 있다.  
* **array**  
유형이나, 그 크기에 따라 달라진다.  
* **pointer**  
유형이나 크기에 관계없이 항상 크기가 동일하다.  

### 산술(arithmatic) 연산 가능 여부 
포인터에서는 산술 연산을 직접 사용할 수 있지만, 배열에서는 포인터로 변환한 후에야 산술 연산이 가능하다.
```Cpp
    int x[3];
    int *y;
    int z;

    y = x;   // OK, int[]는 int*로 암묵적으로 변환됨
    x = y;   // ERROR, 배열 이름은 변경할 수 없음

    z = *x;  // 크래시가 발생하지 않음
    z = *y;  // y가 유효한 주소를 가리키지 않으면 크래시 발생

    y++;     // OK, 포인터는 증가 가능
    x++;     // ERROR, 배열 이름은 증가할 수 없음
```

### 어셈블리 코드 관점
```Cpp
int array[3];
array[2] = 666;
// array[0] 에서 두 칸을 이동하여 그 값을 666으로 변경한다.

int *ptr = new int[3];
ptr[2] = 66;
// ptr[0] 에서 값/주소를 가져오기 위해 주소에 2를 더하고, 역참조를 통해 그 주소에 있는 값에 66을 대입한다.  
```

```Assembly
// For the first code with an integer it is:
2212: main(){
00401000   push        ebp
00401001   mov         ebp,esp
00401003   sub         esp,44h
00401006   push        ebx
00401007   push        esi
00401008   push        edi
2213:     int int_input;
2214:     cin>>int_input;
00401009   lea         eax,[ebp-4]
0040100C   push        eax
0040100D   mov         ecx,offset cin (00414c58)
00401012   call        istream::operator>> (0040b7c0)
2215:     cout<<(int_input+4)<<endl;
00401017   push        offset endl (00401070)
0040101C   mov         ecx,dword ptr [ebp-4]
0040101F   add         ecx,4
00401022   push        ecx
00401023   mov         ecx,offset cout (00414c18)
00401028   call        ostream::operator<< (0040b3e0)
0040102D   mov         ecx,eax
0040102F   call        ostream::operator<< (00401040)
2216:     return 0;
00401034   xor         eax,eax
2217: }
```
```Assembly
// And for the code with pointer it is:
2212: main(){
00401000   push        ebp
00401001   mov         ebp,esp
00401003   sub         esp,4Ch
00401006   push        ebx
00401007   push        esi
00401008   push        edi
2213:     int *int_ptr = new int[1];
00401009   push        4
0040100B   call        operator new (004011b0)
00401010   add         esp,4
00401013   mov         dword ptr [ebp-8],eax
00401016   mov         eax,dword ptr [ebp-8]
00401019   mov         dword ptr [ebp-4],eax
2214:     cin>>*int_ptr;
0040101C   mov         ecx,dword ptr [ebp-4]
0040101F   push        ecx
00401020   mov         ecx,offset cin (00414c38)
00401025   call        istream::operator>> (0040b8a0)
2215:     cout<< (*int_ptr + 4)<<endl;
0040102A   push        offset endl (004010a0)
0040102F   mov         edx,dword ptr [ebp-4]
00401032   mov         eax,dword ptr [edx]
00401034   add         eax,4
00401037   push        eax
00401038   mov         ecx,offset cout (00414bf8)
0040103D   call        ostream::operator<< (0040b4c0)
00401042   mov         ecx,eax
00401044   call        ostream::operator<< (00401070)
2216:     delete(int_ptr);
00401049   mov         ecx,dword ptr [ebp-4]
0040104C   mov         dword ptr [ebp-0Ch],ecx
0040104F   mov         edx,dword ptr [ebp-0Ch]
00401052   push        edx
00401053   call        operator delete (00401120)
00401058   add         esp,4
2217:     return 0;
0040105B   xor         eax,eax
2218: }
```
배열의 값과 포인터가 가리키는 주소를 통한 접근한 값은 다르다. 배열의 값은 정수 값이 저장된 메모리의 일부분인 반면, 포인터는 주소를 저장하는 메모리 공간이기 때문이다. 따라서 컴파일러는 이 두 가지를 번역하는 과정에서 다르게 처리할 수밖에 없다.  
<br>

# 참고
* [[C++] 함수에 배열을 매개변수로 사용할 때 (Passing Arrays To Functions)](https://code-studies.tistory.com/34)
* [Arrays are Not Pointers](https://www.panix.com/~elflord/cpp/gotchas/)
* [Array is not pointer](https://cplusplus.com/forum/articles/10/)