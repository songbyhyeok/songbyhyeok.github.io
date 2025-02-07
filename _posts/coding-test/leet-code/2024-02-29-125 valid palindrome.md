---
title: 125 Valid Palindrome
categories: leet-code
---

# 문제
## [125 Valid Palindrome](https://leetcode.com/problems/valid-palindrome/)
level easy, lang java  
  
**하나의 문장이 팰린드롬인지 확인하는 문제**  

    Input: s = "A man, a plan, a canal: Panama"
    Output: true

    Input: s = "race a car"
    Output: false

    Input: s = " "
    Output: true


<br>

# 특징
* 문자열을 양 끝에서부터 비교하여 같은지 확인할 수 있다.  
* 문자열 활용 메서드를 다양하게 사용할 수 있다.  
<br>

# 시간복잡도
* 투포인터 o(n)  
* 문자열 직접 비교o(n)  
<br>

# 코드  
```
public boolean isPalindrome(String s) {
        int start = 0;
        int end = s.length() - 1;
        // 서로 중앙으로 이동해 나가다 겹치는 지점에 도달하면 종료
        while (start < end) {
            // 영숫자인지 판별하고 유효하지 않으면 한 칸씩 이동
            if (!Character.isLetterOrDigit(s.charAt(start))) {
                start++;
            } else if (!Character.isLetterOrDigit(s.charAt(end))) {
                end--;
            } else { // 유효한 문자라면 앞 글자와 뒷 글자를 모두 소문자로 변경해 비교
                if (Character.toLowerCase(s.charAt(start)) != Character.toLowerCase(s.charAt(end))) {
                    // 하나라도 일치하지 않는다면 팰린드롬이 아니므로 false 리턴
                    return false;
                }
                // 앞쪽 문자는 한 칸 뒤로, 뒤쪽 문자는 한 칸 앞으로 이동
                start++;
                end--;
            }
        }
        // 무사히 종료될 경우 팰린드롬이므로 true 리턴
        return true;
    }
```
```
public boolean isPalindrome(String s) {
        // 정규식으로 유효한 문자만 추출한 다음 모두 소문자로 변경
        String s_filtered = s.replaceAll("[^A-Za-z0-9]", "").toLowerCase();
        // 문자열을 뒤집은 다음 String으로 변경
        String s_reversed = new StringBuilder(s_filtered).reverse().toString();
        // 두 문자열이 동일한지 비교
        return s_filtered.equals(s_reversed);
    }
```