---
title: 택배 배달과 수거하기 레벨 2 - Java
categories: programmers
---

## 목차
- [문제](#문제)
- [풀이](#풀이)
  - [접근 방식](#접근-방식)
  - [Stack 접근](#stack-접근)
  - [Greedy 접근](#greedy-접근)
- [코드](#코드)

# 문제
[https://school.programmers.co.kr/learn/courses/30/lessons/150369](https://school.programmers.co.kr/learn/courses/30/lessons/150369)  
  
**트럭에 상자를 실을 수 있는 크기 Cap개 만큼 싣고 n개의 집을 왕복하여 배달과 수거를 동시에 수행하여 걸린 이동 거리 중 최소 이동 거리를 구하는 문제**  

# 풀이
이동거리를 구할 때 고려할 점은 배달/수거 시 어느 한 쪽이 더 멀다면 거기까지 반드시 수행해야 한다는 점이다. 그리고 효율적으로 일의 양을 줄이기 위해서는 가장 먼 지점의 할당량부터 처리해야 한다.  

    -> 가장 먼 거리 기준 배달/수거 수행 = distance * 2  
    -> 가장 먼 지점부터 수행 = index 접근을 가장 먼 지점부터 진행  

## 접근 방식
## Stack 접근  
1. 가장 먼 지점부터 수행의 조건을 Stack의 LIFO 구조 특징을 잘 활용하여 이행할 수 있다.  
    * ex 1/0/3/1/2 = stack.pop() = 2/1/3/0/1  
2. 가장 먼 거리 기준 배달/수거의 조건을 배달.Stack, 수거.Stack을 통해 구할 수 있다.  
    * ex 배달 1/0/3/1/2, 수거 0/3/0/4/0 = 배달.pop().idx = 4, 수거.pop().idx = 3  
    가장 먼 거리 = (Max(4, 3) + 1) * 2  
3. 해당 지점 할당량 처리할 때 경우의 수를 통해 알고리즘 설계  
    * ex 1/0/3/1/2와 cap = 4일 경우  
    // cap - 2 = 남은 배달양 // 남은 배달양 >= 0 cap -= 배달량  
    그게 아니라면 배달[idx] -= 배달할 수 있는 양만큼만 차감  

## Greedy 접근
1. 가장 먼 거리부터 배달/수거를 하기 위해 배열 접근을 역순으로 진행한다.  
2. 글로 표현하기엔 무리가 된다고 판단, 그림으로 설명하는 것이 맞다고 생각이 들지만, 그냥 간단하게 표현하려고 한다.  
배달/수거의 조건 기준은 현재 수행량 기준에서 해당 지점의 할당량을 처리했을 때 - 상태인지 아닌지로 판단하면 된다.  

|목적|비용|비용|비용|비용|비용|
|--|-|-|-|-|-|
|배달|1|0|3|**1**|**2**|
|수거|0|3|0|**4**|**0**|

![다운로드 (5)](https://github.com/user-attachments/assets/22f2fb49-5be7-4d5c-91f8-55f0df913565)  

**첫 번째 배달 과정**  
[4] 지점을 배달하여 dC = 2, pC = 0 0보다 크므로 dC -= cap(4) = -2, pC = -4  
answer = 현재 지점이 가장 먼 거리이므로 (idx + 1) * 2  
[3] 지점을 배달하여 dC = -1, pC = 0 0보다 크지 않으므로 pass  

이처럼 미리 할당량을 먼저 빌린 다음에 차감 방식으로 접근을 한다면 해당 문제를 풀 수 있다.  

# 코드
```
// Stack 풀이 
import java.util.Stack;

class Solution {
    public long solution(int cap, int n, int[] deliveries, int[] pickups) {
        Stack<Integer> dS = new Stack<>();
        Stack<Integer> pS = new Stack<>();
        long answer = 0;
        
        for(int i = 0; i < n; i++) {
            if (deliveries[i] != 0) {
                dS.push(i);
            }
            if (pickups[i] != 0) {
                pS.push(i);
            }
        }
        
        while(!dS.empty() || !pS.empty()) {
            final int dLen = !dS.empty() ? dS.peek() + 1 : 0;
            final int pLen = !pS.empty() ? pS.peek() + 1: 0;
            answer += Math.max(dLen, pLen) * 2;
            
            int dC = cap;
            int pC = cap;
            
            while(!dS.empty()) {
                final int dIdx = dS.pop();
                final int d = deliveries[dIdx];
                final int calcD = dC - d;
                if (calcD > 0) {
                    dC -= d;
                    continue;
                } else if (calcD == 0) {
                    break;
                }
                
                deliveries[dIdx] -= dC;
                dS.push(dIdx);
                break;
            }
            
             while(!pS.empty()) {
                final int pIdx = pS.pop();
                final int p = pickups[pIdx];
                final int calcP = pC - p;
                if (calcP > 0) {
                    pC -= p;
                    continue;
                } else if (calcP == 0) {
                    break;
                }
                
                pickups[pIdx] -= pC;
                pS.push(pIdx);
                break;
            }
        }
        
        return answer;
    }
}
```

```
// Greedy 풀이
class Solution {
    public long solution(int cap, int n, int[] deliveries, int[] pickups) {
        long answer = 0;
        int dC = 0, pC = 0;
        
        for(int i = n - 1; i >= 0; i--) {
            dC += deliveries[i];
            pC += pickups[i];
            while(0 < dC || 0 < pC) {
                dC -= cap;
                pC -= cap;
                answer += (i + 1) * 2;
            }
        }
        
        return answer;
    }
}
```