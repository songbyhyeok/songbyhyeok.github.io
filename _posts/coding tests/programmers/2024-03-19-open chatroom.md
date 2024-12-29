---
title: 오픈채팅방 레벨 2 - Java
categories: programmers
---

# 문제
[https://school.programmers.co.kr/learn/courses/30/lessons/42888](https://school.programmers.co.kr/learn/courses/30/lessons/42888)  
  
* 오픈채팅방 개설  
* 명령어 ENTER, LEAVE, CHANGE
* 들어갈 때, 변경할 때 기존 ID 변경될 경우 이전 LOG의 ID도 변경  
<br>

# 풀이
## record의 data를 가공하여 nick + command 조합으로 출력하기  
1. **ID 값을 통해 NICK값에 접근할 수 있는 hash에 어떻게 삽입할 것인가?**  
-> hash 삽입 과정에서 leave 명령어는 변동을 줄ㄴ 수 없으므로 넘어가기.  
2. **result 형식으로 조정하여 출력 방법은?**  
-> 출력 과정에서 change 명령어의 Log는 요구하지 않으므로 넘어가기.  
<br>

# 코드
```
import java.util.HashMap;
import java.util.Map;

class Solution {
    public String[] solution(String[] record) {
        String[] answer = null;
        Map<String, String> ids = new HashMap<>();
        int uncountedCnt = 0;
        
        for(String r : record) {
            final String[] info = r.split(" ");
            final char cmdF = info[0].charAt(0);
            if (!(cmdF == 'L')) {
                ids.put(info[1], info[2]);
                
                if (cmdF == 'C') {
                    ++uncountedCnt;
                }
            }
        }
        
        answer = new String[record.length - uncountedCnt];
        int aIdx = -1;
        for(String r : record) {
            final String[] info = r.split(" ");
            final char cmdF = info[0].charAt(0);
            if (cmdF == 'C') {                
                continue;
            }
            
            answer[++aIdx] = ids.get(info[1]) + "님이 " + ((cmdF == 'E') ? "들어왔습니다." : "나갔습니다.");
        }
        
        return answer;
    }
}
```