---
title: 49 Group Anagrams
categories: leet-code
---

# 문제
## [49 Group Anagrams](https://leetcode.com/problems/group-anagrams/description/)
level medium, lang java  
  
**애너그램 단위로 그룹핑**  
* 애너그램?  
-> 문자를 재배열하여 다른 뜻을 가진 단어로 바꾸는 것ex ate는 eat가 될 수도 있고, tea가 될 수도 있다.  
![다운로드 (26)](https://github.com/user-attachments/assets/9cb8a4bb-03d3-41b7-96d4-292aff9cbe3a)  
<br>

# 풀이
책에 풀이 방법을 보기 전, 먼저 어떻게 접근해야 할지 생각을 해봤다.  
  
**How**  
어떻게 했길래? eat, ate, tea는 같아질 수가 있지?  

## 완전탐색
각 요소를 재배치 시키기 위해서는 완전탐색 접근 방법으로 경우의 수를 생각했을 때 3자리 기준 나머지 두 자리를 재배치 시켰을 때 Len * eleLen - 1 * eleLen 의 비용이 발생하게 되고, 이를 또 확인해 주어야 하므로 아니다 싶었다.  
그래서 책 풀이 방법 정렬하여 비교란을 보게 됐고 정렬을 통해 풀이 접근을 했네? 다시 한번 생각해 보고 그다음 모르겠으면 답을 봐야겠다고 전략을 짰다.  
## 정렬
정렬? 정렬을 하게 된다면, 위의 제시된 입력값 eat, tea, tan, ate, ant, cat는 알파벳 순인 a부터 ~ t까지 재구성될 것이다. 그렇게 해서는 같은 애너그램 그룹으로 묶을 수가 없다. 그래서 애너그램 그룹핑된 ate와 eat 그리고 tea를 보았더니 정렬한 이유를 알 수 있었다. 바로 전체 별로 보는 게 아니라 각 요소별로 정렬을 하는 것이다.  
ate, eat, tea를 각 요소별로 정렬 시 ate, ate, ate가 된다. 같은 애너그램이란 것이다. 아! 그렇다면 자료구조 hashMap에 key 값에 이들을 넣고 value에는 이들 요소 문자열을 넣는다면 되지 않을까? 해답을 찾게 되었다.  
<br>

# 알고리즘 특징
정렬과 HashMap을 활용하여 효율적이게 구조화시킨 것  
<br>

# 시간복잡도
![다운로드 (27)](https://github.com/user-attachments/assets/5441317a-fe8f-4462-b7c9-f3d2eba4e39f)  

* Length = 10 ^ 4 = 10,000  
* Element Length = 100  
* Time Complexity = 10,000 * 100 log 100  = 10,000 * 700 = 7,000,000  
<br>

# 코드  
```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> answer = new HashMap<>();

        for(String str : strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String strKey = String.valueOf(chars);
            
            if (!answer.containsKey(strKey)) {
                answer.put(strKey, new ArrayList<>());
            }

            answer.get(strKey).add(str);
        }

        return new ArrayList<>(answer.values());
    }
}
```