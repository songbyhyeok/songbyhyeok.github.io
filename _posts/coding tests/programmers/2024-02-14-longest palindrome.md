---
title: 가장 긴 팰린드롬 레벨 3 - Java
categories: programmers
---

# 문제
[https://school.programmers.co.kr/learn/courses/30/lessons/12904](https://school.programmers.co.kr/learn/courses/30/lessons/12904)  
  
![팰린드롬 문제](https://github.com/user-attachments/assets/242dcfda-9c11-4b29-8d87-8e7dc651ae7c)  

# 풀이
![입출력](https://github.com/user-attachments/assets/c57c2587-dd72-4583-95d6-c17ad5d90439)  

1번 입출력 예 abcdcba는 시작과 끝을 비교하며 줄여가면 되지만, 2번 입출력 예는 시작점과 끝이 같이 줄어드는 방식이 아닌 듯했다. 보면 aba라는 값이 팰린드롬이라 3번 반환한 건데 조금 생각을 해 보니 시작점은 고정이고 비교 대상인 끝점만 유동적으로 줄어든 게 보였다. 그렇다면 끝점만 위치 이동하면 되는가로 생각을 했었는데 "abacedde"란 케이스를 만들어 생각해 보니 역시나 아니었다. 이 케이스는 aba란 3개의 팰린드롬을 만들 수도 있지만 edde란 4개의 팰린드롬 또한 만들 수 있다. 즉, 시작점 또한 고정이 아닌 유동성인 특징을 가졌다는 것이다.  

## 투포인터와 완전탐색
팰린드롬 기초 문제를 풀 때 투포인터를 통해 푸는 것을 알고 있었다. 그리고 선택한 지점 기준에서 가장 많은 팰린드롬을 찾아야 하면서 그 개수가 가장 긴 팰린드롬이라는 보장을 찾기 위해서는 완전 탐색을 통해 계속 찾아야만 했다.  
![다운로드 (2)](https://github.com/user-attachments/assets/d0c2775d-6b6c-418e-ba76-169bee8b2516)  
완전탐색을 통해 i의 위치 char와 j의 위치 char가 같다면 투포인터 알고리즘을 돌려 answer에 가장 긴 length를 math.max를 통해 구하면 된다.  

# 이슈
![다운로드 (3)](https://github.com/user-attachments/assets/d3f8349b-f232-4299-b3d9-d1b62ab55902)  
17 ~ 18번 케이스 실패를 해결하려면 길이 최솟값은 반드시 1이어야 한다.  
-> "abcde"의 팰린드롬 길이는 1이다.  

**효율성 테스트2**  
![다운로드 (4)](https://github.com/user-attachments/assets/56af2da0-cc8e-44bc-98b6-0e4d4ca7e919)  
처음에 시작점 i와 비교 대상은 i + 1의 위치인 j로 시작해 비효율적으로 가장 긴 길이를 구하려고 했기 때문에 실패하게 됐다. 그렇지만 2500개의 문자열을 시간복잡도 분석을 했을 때 2500 * 2499 = 6,247,500 이 나오므로 전혀 문제가 되지 않는데도 실패했다는 점에서 조금 당황스러웠다.  

# 코드  
```
class Solution {
    public static int solution(String s) {
        int answer = 1;
        final int sLen = s.length();
        // 길이가 1인 경우 처리
        if (sLen == 1) {
            return 1;
        }

        for(int i = 0; i < sLen; i++) {
            final char sp = s.charAt(i);
            for(int j = sLen - 1; j > i; j--) {
                // i와 비교 대상인 j가 같다면 two pointer 알고리즘을 수행한다.
                if (sp == s.charAt(j)) {
                    int start = i;
                    int end = j;
                    boolean isSuccess = true;

                    while(start < end) {
                        if (s.charAt(start) != s.charAt(end)) {
                            isSuccess = false;
                            break;
                        }

                        ++start;
                        --end;
                    }

                    if (isSuccess) {
                        answer = Math.max(answer, (j - i) + 1);
                        // 최대 길이를 구하는 문제이기 때문에 그 이하의 값을 찾는 비교는 성능 낭비다.
                        j = i;
                    }                
                }
            }
        }

        return answer;
    }

    public static void main(String[] args) {
        solution("a");
        solution("aa");
        solution("abcdcba");
        solution("abacde");
        solution("abacedde");
    }
}
```