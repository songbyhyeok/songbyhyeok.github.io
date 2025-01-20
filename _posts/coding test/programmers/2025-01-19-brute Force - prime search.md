---
title: 완전탐색 - 소수 찾기 레벨 2 - Java
categories: programmers
---

# Description
## [완전탐색 - 소수 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/42839)  

### 문제설명:
한자리 숫자가 적힌 종이 조각이 흩어져있습니다. 흩어진 종이 조각을 붙여 소수를 몇 개 만들 수 있는지 알아내려 합니다.

각 종이 조각에 적힌 숫자가 적힌 문자열 numbers가 주어졌을 때, 종이 조각으로 만들 수 있는 소수가 몇 개인지 return 하도록 solution 함수를 완성해주세요.  

-> **모든 경우의 수를 모아서, 소수가 몇 개인지 판별하는 문제.**  

### 제한사항:
* numbers는 길이 1 이상 7 이하인 문자열입니다.  
* numbers는 0~9까지 숫자만으로 이루어져 있습니다.  
* "013"은 0, 1, 3 숫자가 적힌 종이 조각이 흩어져있다는 의미입니다.  

### 입출력 예:
![Image](https://github.com/user-attachments/assets/4c009a0d-de09-48f5-b546-985d390fee1d)  

* **예제 #1**  
[1, 7]으로는 소수 [7, 17, 71]를 만들 수 있습니다.  

* **예제 #2**  
[0, 1, 1]으로는 소수 [11, 101]를 만들 수 있습니다.  
  * 11과 011은 같은 숫자로 취급합니다.  
<br>

# Analysis
1. **경우의 수 수집**  
모든 경우의 수를 수집하기 위해서 순열 방식을 채택.  
   *  **중복된 수**  
    Set 자료구조에 저장하여, 이를 과정에서 방지.
2. **소수 판별**  
숫자 길이는 최대 7자리, 최댓값은 9,999,999 이므로, 범위가 큰 숫자 경우에 효율적으로 처리할 수 있는 에라토스테네스의 채 알고리즘을 채택.  
<br>

# Approach
## 에라토스테네스의 채
```Java
boolean[] hasPrimes = new boolean[10000000];
```
소수를 담기 위해 최댓값을 고려한다. (최댓값은 9,999,999)  

## 순열 조합 구하기
```Java
for (int i = 0; i < other.length(); i++) {
            generateDigitPermutations(comb + other.charAt(i), other.substring(0, i) + other.substring(i + 1), ns);
    }
```
순열 방식로 모든 경우의 수를 구하기 위해 위와 같은 재귀 코드를 작성한다.  

![Image](https://github.com/user-attachments/assets/e022f141-3d1d-4dfc-95e8-43edacb70ef2)  

재귀 호출 과정에서 트리 구조를 통해 가능한 경우의 수를 시각적으로 표현할 수 있다.

### 조합된 수 Set에 저장하기
```Java
if (comb != "") {
            ns.add(Integer.valueOf(comb));
        }
```
comb != "" 의도는 가장 첫 번째 comb string의 값은 들어있지 않기 때문이다.

## 코드
```Java
import java.util.HashSet;
import java.util.Set;

class Solution {
    void sieveOfEratosthenes(boolean hasPrimes[]) {
        final int primesSize = hasPrimes.length;
        for (int i = 2; i < primesSize; i++) {
            hasPrimes[i] = true;
        }

        for (int i = 2; i <= Math.sqrt(primesSize); i++) {
            if (hasPrimes[i]) {
                for (int j = i + i; j < primesSize; j += i) {
                    hasPrimes[j] = false;
                }
            }
        }
    }

    void generateDigitPermutations(String comb, String other, Set<Integer> ns) {
        if (comb != "") {
            ns.add(Integer.valueOf(comb));
        }

        for (int i = 0; i < other.length(); i++) {
            generateDigitPermutations(comb + other.charAt(i), other.substring(0, i) + other.substring(i + 1), ns);
        }
    }
    
    public int solution(String numbers) {
        boolean[] hasPrimes = new boolean[10000000];
        Set<Integer> ns = new HashSet<>();
        int answer = 0;

        // 경우의 수 수집
        generateDigitPermutations("", numbers, ns);

        // 소수 수집
        sieveOfEratosthenes(hasPrimes);

        // 소수 판별
        for (int n : ns) {
            if (hasPrimes[n]) {
                ++answer;
            }
        }
        
        return answer;
    }
}
```  
<br>

# 참고
* [[프로그래머스] 소수 찾기 (완전탐색 Lv. 2) - 자바 Java](https://coding-grandpa.tistory.com/81)
