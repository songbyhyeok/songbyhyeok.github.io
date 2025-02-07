---
title: 문자열 압축
categories: programmers
---

# 문제
## [문자열 압축](https://school.programmers.co.kr/learn/courses/30/lessons/60057)
level 2, lang java  
  
**같은 값이 연속해서 나타나는 문자열은 앞의 개수와 반복되는 값으로 변환시켜 짧은 문자열로 압축시키는 알고리즘 구현 문제**  

**예시**  

    aabbaccc -> 2a2ba3c  
    abcabcdede -> x  
    ababcdcdababcdcd -> 2ab2cd2ab2cd (2개 단위 압축)  
    ababcdcdababcdcd -> 2ababcdcd(8개 단위 압축)  
    abcabcdede -> abcabc2de(2개 단위 압축)  
    abcabcdede -> 2abcdede(3개 단위 압축)  

<br>

# 풀이
투포인터 방법을 사용하면 풀 수 있겠다고 생각했다. 입출력 예시를 투포인터 알고리즘을 통해 분석을 해보며 로직을 그려나갔다.  

**입출력 예시 분석**  

    aabbaccc -> 2a2ba3c(1개) 길이 7개  
    ababcdcdababcdcd -> 2ab2cd2ab2cd(2개) 길이 12개  
    ababcdcdababcdcd -> 2ababcdcd(8개) 길이 9개  
    abcabcdede -> 2abcdede(3개기준) 길이 8개  
    abcabcabcabcdededededede -> 4abcdededededede(3개 기준) 길이 16개  
    abcabcabcabcdededededede -> abcabcabcabc4dede(4개기준) 길이 17개  
    abcabcabcabcdededededede -> 2abcabc2dedede(6개 기준) 길이 14개  
    xababcdcdababcdcd -> x 길이 17개  

**분석을 통해 알게된 사실**  
-> 하나의 선택지점 기준에서 움직이는 타겟 지점은 그 개수를 따라 만들어가는 것  
-> 선택지점 개수보다 타겟지점 개수가 작다면 멈추고 가장 작은 길이 반환하기.  

**StringBuilder**  
선택지점과 목표지점을 비교 과정이 끝나면 누적된 문자열을 저장 처리를 해 주어야 하는데, StringBuilder가 제격이었다.  

**Cnt**  
선택지점과 목표지점이 같을 때 누적 횟수를 기입하기 위해 사용하였다.  
 
**로직 흐름**  
1. 두 비교 지점이 같다면 cnt를 누적시킨다.  
2. 같지 않다면 StringBuilder = cnt + 현재지점, cnt = 1, 선택지점 = 목표지점  
3. 모든 비교가 끝났다면 (StringBuilder + 나머지 문자열).length 값과 answer 값을 비교하여 최소값을 answer에 넣는다.  
<br>

# 이슈
![다운로드 (6)](https://github.com/user-attachments/assets/ce2526d4-e106-4301-96fb-ad944ed7a643)  

로직 설계할 때 선택지점과 목표지점의 값이 다를 경우 선택지점 위치를 옮겨서 목표지점 값을 다시 비교해야 하는데 그 부분을 처리하지 않아서 테스트를 통과하지 못했다.  
-> aabbaccc aa와 b는 다르니 선택지점을 b에서 다시 시작해서 baccc와 비교를 시작해야 하는데 그걸 처리하지 않았다.  
멘탈이 찢겨나갔지만 다른 사람들이 어떻게 풀었는지 훑어봤더니 나와 비슷한 절차 방식 흐름대로 다 똑같이 풀고 있었고 선택지점 = 목표지점을 해 주는 코드를 참고하여 실마리를 얻었다!  

![다운로드 (7)](https://github.com/user-attachments/assets/691a2730-c6ce-492f-a87a-1fbabcf1c515)  

테스트 5 실패 이유는 길이 1개의 처리를 해 주지 못해서이다.  
<br>

# 코드
```
class Solution {
    public int solution(String s) {
        final int strLen = s.length();
        // 길이가 1인 경우 
        if (strLen == 1) {
            return 1;
        }

        int answer = 1000;
        StringBuilder combStr = new StringBuilder();
        
        for (int i = 0; i < strLen; i++) {
            final int len = i + 1;
            String sp = s.substring(0, len);
            int cnt = 1;
            int j = len;

            // 현재 가리키는 길이가 절반 아래일 경우에만
            if (len > strLen / 2) {
                break;
            }

            for (; j < strLen; j += len) {

                // 비교 지점 길이가 범위를 넘어설 경우
                if (j + len > strLen) {
                    break;
                }

                final String ep = s.substring(j, j + len);
                // 두 값이 같다면
                if (sp.equals(ep)) {
                    ++cnt;
                } else { // 같지 않다면
                    // combStr에 문자열을 누적
                    // cnt가 2개 이상일 때 cnt도 누적
                    final String cntStr = 1 < cnt ? Integer.toString(cnt) : "";
                    combStr.append(cntStr + sp);

                    // ep 지점에서 다시 연산 처리를 하기 위한 코드
                    cnt = 1;
                    sp = ep;
                }
            }

            // combStr에 문자열을 누적
            // cnt가 2개 이상일 때 cnt도 누적
            final String cntStr = 1 < cnt ? Integer.toString(cnt) : "";
            combStr.append(cntStr + sp);
            final int compactLen = (combStr.toString() + s.substring(j, strLen)).length();
            answer = Math.min(answer, compactLen);

            // 조합 문자열 초기화
            combStr.setLength(0);
        }

        return answer;
    }
}
```