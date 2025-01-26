---
title: 739 Daily Temperatures 
categories: leet-code
---

# 문제
## [739 Daily Temperatures](https://leetcode.com/problems/daily-temperatures/description/)
level medium, lang java  
  
**주어진 temperatures 리스트를 입력받아 각 요소 기준에서 그다음의 입력 값들과 비교하여 더 따뜻한 날이 존재한다면 며칠이 걸리는지 출력란에 기입하기**  
<br>

# 풀이
## 스택을 활용한 누적치 기입
주어진 [23, 24, 25, 21, 19, 22, 26, 23] 온도 리스트에서 0번 idx 23도와 비교해서 1번 idx 24도는 1도가 더 크기 때문에 23도 기준으로 다음 날에 더 높기 때문에 1일을 기입하면 된다. 이는, 이전과 현재의 값을 비교할 때 스택을 사용하면 적절하다.  
23도를 스택 삽입 -> 현재 24도와 비교하여 더 낮기 때문에 24도의 idx - 23도의 idx를 통해 얻어진 기간 차를 23도의 idx의 출력표란에 기입하면 된다. idx 2번인 25도 기준에서 더 높은 온도는 idx 6번에 26도이다.  
  
**그렇다면 어떻게 도출할 수 있을까?**  
25도를 스택에 삽입 후 연달아 입력된 21도와 19도는 작으므로 계속 스택에 삽입이 된다. 그 다음 22도는 19도 보다 높으므로 19도 기준에서 하루 기간을 출력란에 기입하게 되고 스택에서 제거가 된다. 그다음 삽입됐던 21도가 다시 22도와 비교해서 낮으므로 두 인덱스 차가 2일이 되므로 이 역시 기입과 동시에 스택에서 제거가 된다. 마지막 삽입된 25도는 비교해서 여전히 높으므로 22도가 스택에 삽입되고 26도와 비교해서 낮으므로 4일차를 기입 후에 스택에서 제거가 되는 메커니즘 방식으로 작동하게 되는 구조이다.  
<br>

# 코드  
```
class Solution {
    public int[] dailyTemperatures(int[] temperatures) {        
        var answer = new int[temperatures.length];
        if (1 < temperatures.length) {
            var stack = new ArrayDeque<>(Arrays.asList(0));

            for(int i = 1; i < temperatures.length; i++) {
                while(!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
                    final var lastIdx = stack.pop();
                    answer[lastIdx] = i - lastIdx;
                }

                stack.push(i);
            }
        }

        return answer;
    }
}
```