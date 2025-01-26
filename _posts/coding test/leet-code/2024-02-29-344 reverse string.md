---
title: 344 Reverse String 
categories: leet-code
---

# 문제
## [344 Reverse String](https://leetcode.com/problems/reverse-string/)
level easy, lang java   

  
    Input: s = ["h","e","l","l","o"]
    Output: ["o","l","l","e","h"]
    
뒤집으면 된다.  
<br>

# 풀이
![다운로드 (44)](https://github.com/user-attachments/assets/a3724c06-afe1-4a3d-bae5-d860f5b022e7)  
투 포인터 방식을 활용하여 시작점과 끝점을 이용해 두 단어를 서로 교환하는 과정에서 temp 변수를 사용했다.  
<br>

# 특징
투포인터 방식과 임시 변수 활용을 통해 두 요소를 교환하는 효율적인 알고리즘이다.  
<br>

# 시간복잡도
o(log n)  
<br>

# 코드  
```
class Solution {
    public void reverseString(char[] s) {
        int left = 0;
        int right = s.length - 1;

        while(left < right) {
            char temp = s[left];
            s[left] = s[right];
            s[right] = temp;

            ++left;
            --right;
        }
    }
}
```