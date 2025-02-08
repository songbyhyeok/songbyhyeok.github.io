---
title: 2775번 부녀회장이 될테야
categories: baekjoon
---

## 백준 2775번 - 부녀회장이 될테야
-> 주어진 아파트에서 k층 n호에 살고 있는 사람 수를 계산하는 문제로, 각 층의 사람 수는 아래 층의 사람 수 합계에 따라 결정된다.  

### 문제 조건
* a층의 b호에 사는 사람 수는 (a-1)층의 1호부터 b호까지의 사람들의 합계이다.
* 아파트 층수는 0층부터 있고, 각 층은 1호부터 시작한다, i층에는 i명이 산다.

### 문제 유형
* 다이나믹 프로그래밍

## 풀이 도출 과정
![Image](https://github.com/user-attachments/assets/84e14153-98a0-4a1e-8467-6ab5d6b52f78)  
![Image](https://github.com/user-attachments/assets/cbc8be11-1481-4af6-896c-a0d9a95e6952)  

1. 문제 조건에 맞게 '층'과 '호'를 시각적으로 표현한 후, 그 안에서 일정한 패턴을 파악한다.
2. 동적 계획법(DP)을 사용하여 상향식(bottom-up)과 하향식(top-down) 방식으로 접근할 수 있도록 수식을 작성한다.

## 시간복잡도
* 상향식(Bottom-Up) k*n
* 하향식(Top-Down) 2^N

## 코드
```Java
private void init(int dp[][], final int k, final int n) {
        for(int i = 0; i <= k; i++) {
            dp[i][0] = 1;
        }

        for(int i = 0; i <= n; i++) {
            dp[0][i] = i + 1;
        }
    }

// Bottom-Up
private int tabulate(int dp[][], final int k, final int n) {        
        for(int i = 1; i <= k; i++){
            for(int j = 1; j <= n; j++) {
                dp[i][j] = dp[i][j - 1] + dp[i - 1][j];
            }
        }

        return dp[k][n];
    }

// Top-Down
private int memoize(int dp[][], final int k, final int n) {
        if (dp[k][n] == 1 || k == 0) {
            return dp[k][n];
        }
       
        dp[k][n] = memoize(dp, k, n - 1) + memoize(dp, k - 1, n);
        return dp[k][n];
    }
```

