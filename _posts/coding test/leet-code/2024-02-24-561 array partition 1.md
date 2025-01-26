---
title: 561 Array Partition 1 
categories: leet-code
---

# 문제
## [561 Array Partition 1](https://leetcode.com/problems/array-partition/)
level easy, lang java   
  
**n 크기의 배열이 주어질 때, pair a, b의 min의 합으로 만들 수 있는 가장 큰 수를 출력**  

    [1, 3, 4, 2]min(1, 2) + min(3, 4) = 4  

<br>

# 풀이
## 모든 경우의 수를 그려보기
min(1, 2) + min (3, 4) = 4min(1, 3) + min(2, 4) = 3min(1, 4) + min(2, 3) = 3  
어떻게하면 가장 최댓값을 만들 수 있을까? 가장 작은 값 1과 2를 버리고 3과 4를 더한 7을 만들 수 없을까?  
모든 경우의 수를 계산해 보며 깨달았다. 오름차순 정렬 이후 pair의 구성 중 낮은 값의 합이 가장 크다는 것을

## List 풀이
책에서 List 풀이도 제공하는데, 굳이? Queue나 Stack을 차라리 쓰는 게 더 의미 있지 않을까?  
자료구조의 동작 원리를 이해하는 데는 좋다고 생각은 들긴 했지만 굳이 해당 방식을 통해 풀진 않았다.  
<br>

# 알고리즘 특징
정렬을 통한 배열요소 접근  
<br> 

# 시간복잡도
O(n)  
<br>  

# 코드  
```
class Solution {
    public int arrayPairSum(int[] nums) {
        int answer = 0;

        Arrays.sort(nums);

        for(int i = 0; i < nums.length; i++) {
            if (i % 2 == 0) {
                answer += nums[i];
            }
        }

        return answer;
    }
}
```