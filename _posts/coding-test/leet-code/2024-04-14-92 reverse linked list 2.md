---
title: 92 Reverse Linked List 2
categories: leet-code
---

# 문제
## [92 Reverse Linked List 2](https://leetcode.com/problems/reverse-linked-list-ii/description/)
level medium, lang java  
  
**left와 right까지의 노드를 역순으로 바꾸어 반환하기.**  
```
    ex if 1 -> 2 -> 3 -> 4 -> 5 then 1 -> 4 -> 3 -> 2 - > 5  
```

<br>

# 풀이
## 역순으로 어떻게 바꿀 것인가?
**나의 생각**   
주어진 left와 right 숫자에 맞게 두 노드의 위치를 이동시켜 교환하는 방법인 즉, 투 포인터 교체 방식을 생각했었다.  
-> ex start 1, end 5라면 1과 5를 서로 교체하고 다시 start 2, end 4 재지정하는 방식

**시작 지점 기준으로 순회하기**  
위 투포인터 방식과는 다르게 start -1의 위치를 시작 노드로 end 위치는 start의 위치로 두어 두 노드의 next 부분만 바꾸는 방식으로 진행하게 된다.  
예를 들어 1 2 3 4 5 연결리스트가 주어졌고, left가 1 그리고 right가 3이라면 start는 left - 1 인 0이 되고, right는 start 지점인 1로 지정이 된다. 그리고 교체 방식은 start의 next 1과 end의next는 end의 nextnext를 교체시켜 지정된 start는 고정되어 있고, end만 계속해서 밀리면서 최종적으로 거꾸로 뒤집혀지게 되는 방식이다.  
설명으로만 들을 경우 이해를 못할 수 있으니 숫자가 규칙적으로 어떻게 바뀌는지 보면 이해가 될 것이라고 생각한다.  

    // ex left가 1, right가 5라면  
    0 -> 1 -> 2 -> 3 -> 4 - > 5  
    0 -> 2 -> 1 -> 3 -> 4 - > 5  
    0 -> 3 -> 2 - > 1 -> 4 -> 5  
    0 -> 4 -> 3 - > 2 -> 1 -> 5  
    0 -> 5 -> 4 - > 3 -> 2 -> 1  

지정된 start 포인터 0은 계속 그자리에 있고, end 포인터 1은 계속해서 한 칸 씩 밀리는 것을 알 수 있고. end 포인터가 대각선 형태를 그리는 것을 파악할 수 있다.  
<br>

# 코드  
```
// 순회하며 지정된 두 노드를 바꾸기.
class Solution {
    public ListNode reverseBetween(ListNode head, int left, int right) {
        final var dummy = new ListNode(0);
        var preSt = dummy;
        var end = head;

        dummy.next = head;

        for(int i = 0; i < left - 1; i++) {
            preSt = preSt.next;
        }

        end = preSt.next;

        for(int i = 0; i < right - left; i++) {
            final var relay = preSt.next;
            preSt.next = end.next; 
            end.next = end.next.next;            
            preSt.next.next = relay;
        }

        return dummy.next;
    }
}
```