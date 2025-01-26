---
title: 3차 파일명 정렬
categories: programmers
---

# 문제
## [3차 파일명 정렬](https://school.programmers.co.kr/learn/courses/30/lessons/17686)
level 2, lang java  
  
1. 카카오 입사한 무지가 저장소 서버 관리를 맡게 되었는데 형상관리 버전 기록을 이름순으로 정렬 시 가독성 문제로 문자, 숫자 오름차순 정렬 방식으로 변경을 해야 한다.  
2. 파일은 [] 최대 1000개각각의 파일들은 100글자 이하, 소대문자, 숫자, 공백, ., -만 가지거나 없을 수 있다.  
3. 요구조건* 각 파일을 Head, Number, Tail 세 개로 구조화 시킨다.* 각 요소마다 요구조건에 맞게 정렬 한다.Head대소문자 구별하지 않고 사전순 정렬Number두 문자열이 같다면 오름차순 정렬Tail숫자까지 같다면 정렬하지 않는다.  
<br>

# 풀이
1. 우선 문제 내용이 길다 보니 분석 하는 데 조금 시간이 걸렸다, 위와 같이 구조화시켜 이해하였다.  
2. Java Sort에 대한 개념이 없다 보니 코딩테스트 정렬할 때 사용하는 Comparator와 CompareTo 그리고 람다를 사용하게 되는 것을 알게 되어 해당 기술들을 사용해 정렬했다.  
3. 문자열을 Head, Number, Tail로 구조화 시킨 후 정렬 요구조건에 맞게 Sort 하는 것에 의문점이 있어 난감했었다.  
* Split를 사용하여 Head, Number, Tail로 분리해야 하는데 앞이 문자열일 때 혹 숫자일 때를 구분하여 어떻게 분리할 수 있을까? -> ChatGPT에게 물어봐서 긍정형 전방탐색 정규화 방식을 알려주어 해당 방법을 사용하여 구조화 시켰다.  
* HEAD 문자열 정렬할 때 대소문자를 꼭 구분해야 하는가? 의문이 있어서 이역시 chatGPT에게 물어보니 반드시 그래야 한다고 알려주었고 소문자로 변경 후 compareTo로 비교하였다.  
* Number는 문자열 타입이기 때문에 정렬 크기 비교가 가능할까란 의문에 대해 '0'과 '1'을 비교 시 '0'이 더 크다는 것을 알게 되었고, parseInt 메소드를 사용하여 숫자로 변환 후 비교하였다.  
* 마지막 Head와 Number까지 같을 때 정렬을 하지 말라는 조건은 Number 비교할 때 compare 값이 0이 나와 두 숫자가 같다면 내부적으로 sort가 정렬 처리를 하지 않는다는 것을 알게 되었고, 굳이 조건 처리를 넣지 않았다.  
  
해당 문제를 풀며 문제 이해 분석과 정렬 메커니즘을 이해한다면 추후에는 빠른 속도로 풀 수 있겠다란 생각이 들었고, 무엇보다도 빠른 속도로 풀기 위해서는 정규화 기술을 잘 활용해야 한다고 생각이 들었다.  
<br>

# 이슈
1. 문자열과 숫자 기준으로 구분하기 위해 split 메소드를 사용한다면, 정규화를 어떻게 사용해야 할까?  
2. Head에 대소문자 구별없이 문자열 비교가 가능할까?  
3. Number 앞에 0이 있다면 문자열 크기 비교가 가능할까?  
4. Sort 내부 메커니즘을 정확히 알지 못하는데 Comparator와 CompareTo 메소드를 활용하여 어떻게 조건 정렬을 할 수 있을까?  
<br>

# 다른 풀이  
Pattern 객체를 활용한 정규식 표현법으로 더 깔끔하게 풀 수가 있다. 정규식과 Pattern 객체에 대해서 공부를 해야겠다.  
<br>

# 코드
```
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

class Solution {
    public String[] solution(String[] files) {
        List<String> answer = new ArrayList<>(Arrays.asList(files));
        
        // 0. 두 문자열을 정렬하기 위해 sort comparator 사용
        answer.sort((s1, s2) -> {
            // 1. split를 사용한 정렬
            // Head, Number, Tail로 각각 분리하여 구조화
            // 긍정형 전방탐색 positive lookbehind chat gpt가 알려줌
            // 숫자나 문자 기준으로 분리할 수 있게 해준다.
            String[] s1Parts = s1.split("(?<=\\D)(?=\\d)|(?<=\\d)(?=\\D)");
            String[] s2Parts = s2.split("(?<=\\D)(?=\\d)|(?<=\\d)(?=\\D)");

            // 2. Head는 문자열 비교 (통일하기 위해 소문자 변환 후 진행)
            final int comparedHead = s1Parts[0].toLowerCase().compareTo(s2Parts[0].toLowerCase());
            // 2-1 Number에서 크기 비교 (두 문자열이 같을 때, 숫자 크기 비교하기 위해 숫자 변환 후 진행)
            if (comparedHead == 0) {
                return Integer.compare(Integer.parseInt(s1Parts[1]), Integer.parseInt(s2Parts[1]));
            }

            // 2-2 같지 않기 때문에 문자열 사전순 기준으로 정렬
            return comparedHead;
        });

        return answer.stream().toArray(String[]::new);
    }
}
```