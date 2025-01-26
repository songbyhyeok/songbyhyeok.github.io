---
title: 316 Remove Duplicate Letters 
categories: leet-code
---

# 문제
## [316 Remove Duplicate Letters](https://leetcode.com/problems/remove-duplicate-letters/)
level medium, lang java  
  
**구성된 문자가 중복일 경우 사전 순으로, 아니라면 현재 위치로 두어 문자열 재구성하기**  
<br>

# 풀이
**'bcabc'**  
a는 중복된 문자가 아니기 때문에 현재 위치를 고수하고, bc는 각각 중복되어 있기 때문에 앞의 bc는 지워진다.  
-> 따라서 'abc'다.  
**'cbacdcbc'**  
b와 c는 중복됐기 때문에 앞의 부분을 지운다, a와 d는 중복되지 않아 현재 위치를 고수한다.  
-> 결과: 'acdcbc'  

**상태 보고**  
b는 중복된 값이 없어 다음 위치에 고정되어 있다. 여전히 c는 아직 여러 개가 존재하고 있고, 사전 순의 조건을 고려해야 하기 때문에 다시 한번 중복 처리할 때 어느 위치에 있어야 사전순을 최대한 지킬 수 있는지 판단해야 한다.  

**시뮬레이션**    
* **가장 마지막의 c를 선택한다.**  
-> 'adbc'  
사전 순의 규칙을 위배한다, c의 위치는 중간 or 첫 번째 위치하는 게 더 나은 선택이었다.  
* **중간의 c를 선택한다.**  
-> 'adcb'  
역시 사전 순의 규칙을 위배한다. c의 위치를 첫 번째로 선택했더라면 사전 순대로 나열하였다.  
* **첫 c를 선택한다.**  
-> 'acdb'  
위처럼 단순히 중복된 문자들을 모두 처리하고, 사전 순대로 하라는 규칙은 자세히 분석하면 추상적인 지시라는 것을 알 수 있다. 그렇기 때문에 지문 수준과 구현 과정의 코드의 난이도가 높을 수 밖에 없는 것이다. 
 
**Stack 방식**  
'case cbacdcbc'를 토대로 하나의 문자마다 중복인지 확인을 해야 하고 만약, 맞다면 사전 순의 규칙을 고려해야 한다는 사실을 체득하게 되었다.  
    
**'dbdcacab'**  
1. **'d'는 중복된 값이고, 다음 문자와 비교하여 크기 때문에 배제.**  
2. **'b'는 중복 값이지만, 다음 값보다 작기 때문에 고정, 따라서 외의 동일 'b'들은 모두 배제**  
3. **'d'는 중복 x이기 때문에 고정**  
4. **'c'는 중복됐고, 다음 값보다 크기 때문에 배제**  
    임시 결과 -> 'xbdxacab'  
5. **'a'는 중복됐지만, 다음 값보다 작기 때문에 고정. 따라서 외의 동일 'a'들은 모두 배제**  
6. **'c'는 중복 x이기 때문에 고정**  
7. **'a'는 배제됐기 때문에 x**  
8. **'b'는 배제됐기 때문에 x**  
9. **결과는 -> 'xbdxacxx' -> 'bdac'**  
  
여러 개의 동일 문자들은 어느 위치에 있는지에 따라 사전 순을 위배할 수도 아닐 수도 있다. 따라서 Recursive의 방식으로도 풀이가 가능하다.  
  
**Recursive 방식**  
  
**'dbdcacab'**  
1. **주어진 문자열(A) 구성의 중복을 제거 및 정렬한 구성 B를 만든다.**  
2. **B의 첫 문자를 이전의 A의 위치에 대조하여 일치한 문자열 구성 C를 만든다.**   
3. **B와 C가 일치한다면 B의 첫 문자는 해당 위치에 고정시키고 B를 바탕으로 다시 1 to 3의 과정을 하나의 문자가 남을 때까지 완수 및 통합한다.**  
<br>

# 코드  
```
// Recursive
class Solution {
    private Set<Character> toSortedSet(final String s) {
        // Set<Character> cSet = new TreeSet<>( new Comparator<Character>() {
        //     @Override
        //     public int compare(final Character c1, final Character c2) {
        //         return c1 == c2 ? 0 : c1 > c2 ? 1 : -1;
        //     }
        // });

        Set<Character> cSet = new TreeSet<>( (o1, o2) -> {
            return o1.compareTo(o2);
        });

        for(var c : s.toCharArray()) {
            cSet.add(c);
        }

        return cSet;
    }

    public String removeDuplicateLetters(String s) {
        if (1 < s.length()) {
            final var cSet = toSortedSet(s);
            for(var c : cSet) {
                final var suffixS = s.substring(s.indexOf(c));
                if (cSet.equals(toSortedSet(suffixS))) {
                    return c + removeDuplicateLetters(suffixS.replace(String.valueOf(c), ""));
                }
            }
        }

        return s;
    }
}
```

```
// Stack
class Solution {
    public String removeDuplicateLetters(String s) {
        Map<Character, Integer> counter = new HashMap<>();
        Map<Character, Boolean> seen = new HashMap<>();
        Deque<Character> stack = new ArrayDeque<>();

        for(var c : s.toCharArray()) {
            final var nullableN = counter.get(c);
            counter.put(c, nullableN == null ? 1: nullableN + 1);
        }

        for(var c : s.toCharArray()) { 
            counter.put(c, counter.get(c) - 1);

            final var seenStatus = seen.get(c);
            if (seenStatus != null && seenStatus == true) {
                continue;
            }

            while(!stack.isEmpty() && stack.peek() > c && counter.get(stack.peek()) > 0) {
                seen.put(stack.pop(), false);
            }

            stack.push(c);
            seen.put(c, true);
        }

        var sb = new StringBuilder();
        while(!stack.isEmpty()) {
            sb.append(stack.pollLast());
        }

        return sb.toString();
    }
}
```