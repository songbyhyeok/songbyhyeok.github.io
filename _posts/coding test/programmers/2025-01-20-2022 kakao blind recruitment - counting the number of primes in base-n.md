---
title: 2022 KAKAO BLIND RECRUITMENT - k진수에서 소수 개수 구하기 레벨 2 - Java
categories: programmers
---

# Description
## [KAKAO BLIND RECRUITMENT - k진수에서 소수 개수 구하기](https://school.programmers.co.kr/learn/courses/30/lessons/92335)  

### 문제 설명:
양의 정수 n이 주어집니다. 이 숫자를 k진수로 바꿨을 때, 변환된 수 안에 아래 조건에 맞는 소수(Prime number)가 몇 개인지 알아보려 합니다.

0P0처럼 소수 양쪽에 0이 있는 경우
P0처럼 소수 오른쪽에만 0이 있고 왼쪽에는 아무것도 없는 경우
0P처럼 소수 왼쪽에만 0이 있고 오른쪽에는 아무것도 없는 경우
P처럼 소수 양쪽에 아무것도 없는 경우
단, P는 각 자릿수에 0을 포함하지 않는 소수입니다.
예를 들어, 101은 P가 될 수 없습니다.
예를 들어, 437674을 3진수로 바꾸면 211020101011입니다. 여기서 찾을 수 있는 조건에 맞는 소수는 왼쪽부터 순서대로 211, 2, 11이 있으며, 총 3개입니다. (211, 2, 11을 k진법으로 보았을 때가 아닌, 10진법으로 보았을 때 소수여야 한다는 점에 주의합니다.) 211은 P0 형태에서 찾을 수 있으며, 2는 0P0에서, 11은 0P에서 찾을 수 있습니다.

정수 n과 k가 매개변수로 주어집니다. n을 k진수로 바꿨을 때, 변환된 수 안에서 찾을 수 있는 위 조건에 맞는 소수의 개수를 return 하도록 solution 함수를 완성해 주세요.

-> **10진법 n을 k진수로 변환 이후, 주어진 조건에 맞는 소수를 찾아 그 개수를 반환하는 문제.**  

### 제한사항:
* 1 ≤ n ≤ 1,000,000  
* 3 ≤ k ≤ 10  

### 입출력 예:
![Image](https://github.com/user-attachments/assets/d7fea4d2-c450-40ba-a6aa-ced7c53852ea)  

* **예제 #1**  
문제 예시와 같습니다.  

* **예제 #2**  
110011을 10진수로 바꾸면 110011입니다. 여기서 찾을 수 있는 조건에 맞는 소수는 11, 11 2개입니다. 이와 같이, 중복되는 소수를 발견하더라도 모두 따로 세어야 합니다.  
<br>

# Analysis
1. n (10진수) 값을 k 진법으로 변환
2. 에라토스테네스의 체를 이용한 소수 필터링
   1. n의 최댓값인 1,000,000을 6진수로 변환한 결과 33,233,344가 도출되므로, bool 배열의 크기는 최대 8자리로 지정.
3. 에라토스테네스의 체를 활용하여 요구된 조건에 맞는 소수의 개수 세기

<br>

# Approach
## 진법 변환 알고리즘
```Java
private String convertToBaseK(final int n, final int k) {
        StringBuilder cN = new StringBuilder();
        int copiedDecimal = n;
        while(copiedDecimal != 0) {
            cN.append(copiedDecimal % k);
            copiedDecimal /= k;
        }
        
        return cN.reverse().toString();
    }
```
변환된 숫자들은 문자열로 순차적으로 저장한 후, 다시 역으로 변환한다.

## 소수 판별 알고리즘
```Java
private boolean isPrime(final Long n) {
        if (n == 1) {
            return false;
        }
            
        if (n == 2 || n == 3 || n == 5 || n == 7 || n == 11) {
            return true;
        }
            
        for(int i = 2; i <= Math.sqrt(n); i++) {
            if (n % i == 0) {
                return false;
            }
        }
        
        return true;
    }
```
제곱근을 이용하여 끝까지 판별하지 않고, 효율적으로 일정 범위까지만 판별한다.

## Issue
### 에라토스테네스의 체 core dumped
4byte가 저장할 수 있는 범위를 초과하여 힙 오버플로우가 발생하므로, 이를 해결하기 위해 최적화된 소수 판별 알고리즘으로 대체해야 한다.
### 테스트 케이스 1, 11 core dumped
변환된 10진수 값이 4byte 범위를 초과할 때 발생하는 문제로, 이를 해결하기 위해 8byte 범위를 표현할 수 있는 Long 타입으로 변경해야 함.

## 코드
```Java
class Solution {
    private String convertToBaseK(final int n, final int k) {
        StringBuilder cN = new StringBuilder();
        int copiedDecimal = n;
        while(copiedDecimal != 0) {
            cN.append(copiedDecimal % k);
            copiedDecimal /= k;
        }
        
        return cN.reverse().toString();
    }
    
    private boolean isPrime(final Long n) {
        if (n == 1) {
            return false;
        }
            
        if (n == 2 || n == 3 || n == 5 || n == 7 || n == 11) {
            return true;
        }
            
        for(int i = 2; i <= Math.sqrt(n); i++) {
            if (n % i == 0) {
                return false;
            }
        }
        
        return true;
    }
    
    private int countPrimes(final String cvtN) {
        StringBuilder cN = new StringBuilder();
        int answer = 0;
        int i = 0;
        boolean isPrimeCheckNecessary = false;
        while(!(i >= cvtN.length() && cN.length() == 0)) {
            if (i < cvtN.length()) {
                final char selectedN = cvtN.charAt(i);
                ++i;
                                
                if (selectedN != '0') {
                    cN.append(selectedN);
                    isPrimeCheckNecessary = false;
                } else {
                    isPrimeCheckNecessary = true;
                }                                     
            } else {
                isPrimeCheckNecessary = true;
            }
            
            if (isPrimeCheckNecessary) {
                if (cN.length() != 0) {
                    if (isPrime(Long.parseLong(cN.toString()))) {
                        ++answer; 
                    }
                }                

                cN.setLength(0);
                isPrimeCheckNecessary = false;
            }
        }   
        
        return answer;
    }
    
    public int solution(int n, int k) {
        return countPrimes(convertToBaseK(n, k));
    }
}
```
