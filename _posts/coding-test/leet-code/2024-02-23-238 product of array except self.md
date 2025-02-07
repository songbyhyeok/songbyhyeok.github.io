---
title: 238 Product of Array Except Self
categories: leet-code
---

# 문제
## [238 Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/description/)
level medium, lang java  
  
**nums[i]인 자신을 제외한 나머지 nums 모든 요소의 곱셈 결과가 되도록 출력을 요구하는 문제**  

    input -> [1, 3, 5, 7]  
    output -> [105, 35, 21, 15]  

나눗셈을 하지 않고 O(n)에 반드시 풀이할 것  
<br>

# 풀이  
## 나눗셈 풀이 
반복문 -> nums 모든 요소를 곱한 값 / nums[i]를 할 경우 o(n)으로 풀 수가 있다.  

## 자기 자신을 제외한 곱셈 결과를 통한 해결
자바 알고리즘 인터뷰 책에선 자신을 제외, 순차적인 왼쪽의 곱셈 결과, 오른쪽 곱셈 결과를 배열로 저장 풀이를 제공한다.  
해당 풀이 기법명은 책에서 언급되진 않았지만, Prefix Sum 기법을 착안한 방식이다. 하지만 구간합 개념을 아직 다루지 못하는 입장에서 풀이 방법을 제공한다고 한들 이해가 정말 되지 않았고 왜 이 방법을 어떻게 생각했길래 풀이 방법으로 사용할 수 있었을까에 중점적으로 몰두하였다. 그리고 그 이유를 알 수 있었다.  
![다운로드 (8)](https://github.com/user-attachments/assets/211b06ea-2890-4042-ac76-11bd2401779a)  

**Left 관점**  
![다운로드 (9)](https://github.com/user-attachments/assets/00f25828-ed8d-432f-a007-d8841497affd)  
구간 곱 방식으로 누적된 값들이 순차적으로 배열에 차곡 차곡 쌓여있다.  
위 그림에서 마지막 l의 4번 요소는 곱에 누적시키지 않는다. 그 이유는 마지막 요소이기 때문에 자기 자신을 포함하지 않게 하기 위함이다.  

**Right 관점**  
![다운로드 (10)](https://github.com/user-attachments/assets/a735da3f-3d67-43f7-907b-737b580d4571)  
역순인 Right 배열에는 되로 첫 번째 요소를 포함하지 않는다. 

**두 관점을 통한 이해**  
![다운로드 (11)](https://github.com/user-attachments/assets/5c12b0f4-6ea3-42dd-b3f4-2fb17f5bbc96)  
즉 I의 요소를 제외한 모든 element의 곱을 가지기 위해선 L의 이전 모든 요소의 곱과 R의 이후 모든 요소의 곱을 곱한다면 자기 자신을 제외한 곱 값을 구할 수 있다!  
<br>

# 알고리즘 특징
Prefix Sum 기법 착안한 곱 방식 알고리즘  
<br>

# 시간복잡도
o(n)  
<br>

# 코드  
```
class Solution {
    public int[] productExceptSelf(int[] nums) {
        int[] answer = new int[nums.length];
        int[] lArr = new int[nums.length];
        int[] rArr = new int[nums.length];
        int l = 0;
        int r = nums.length - 1;

        lArr[l++] = 1;
        rArr[r--] = 1;

        while(l < nums.length || r > -1) {
            lArr[l] = lArr[l - 1] * nums[l - 1];
            ++l;

            rArr[r] = rArr[r + 1] * nums[r + 1];
            --r;
        }

        for(int i = 0; i < nums.length; i++) {
            answer[i] = lArr[i] * rArr[i];
        }
        
        return answer;
    }
}
```