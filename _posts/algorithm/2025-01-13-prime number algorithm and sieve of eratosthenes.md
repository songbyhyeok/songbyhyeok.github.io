---
title: 소수 판별 알고리즘, 에라토스테네스의 체
categories: algorithm
---

# 소개
소수가 무엇이고 그리고 어떤 알고리즘이 있는지 다루어보려고 한다.
## 소수
```
1과 100 사이에는 총 25개의 소수가 있다.  
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
```
소수는 1과 그 자체로만 나누어지는 자연수를 말한다.  

### 소수의 특징
* 모든 소수는 2를 제외하고 홀수
* 모든 소수는 1보다 크며, 항상 자연수
* 소수는 무한   

## 합성수
1보다 큰 자연수 중에서 소수가 아닌 수를 말한다.  

### 합성수의 특징
* 합성수는 홀수 or 짝수이다.  
* 모든 합성수는 1과 그자체 이외의 인수를 가져야 한다.(2개 이상의 인수)  
* 약수의 개수가 3개 이상이고 둘 이상의 소수를 곱한 자연수   
<br>

# 소수 판별법
## 기본 분할 방법
2부터 n-1까지 모든 숫자를 확인하면 되는 가장 간단한 방법이다.

### 코드
```Java
static boolean isPrime(int n) {
        if (n <= 1)
            return false;

        // Check divisibility from 2 to n-1
        for (int i = 2; i < n; i++)
            if (n % i == 0)
                return false;

        return true;
    }
```

### 시간복잡도
O(n)

## √(N)(제곱근)까지 확인하는 방법
N의 약수 m과 N/m인 몫이면서 대응되는 약수 q. 두 개를 비교하면 다음과 같다.  
약수 m은 √N 보다 작거나 같다면 대응되는 약수 몫 q는 크거나 같게 되는데, 이때 약수 m만 확인하면 소수를 빠르게 구할 수 있다.  
즉, √N 보다 작거나 같은 약수 m까지만 계산하면 빠르게 소수인지 판별이 가능하다.

### 원리
```
[1 * 1000] [2 * 500] [4 * 250] [5 * 200] [8 * 125] [10 * 100] [20 * 50] [25 * 40]
[40 * 25] [50 * 20] [100 * 10] [125 * 8] [200 * 5] [250 * 4] [500 * 2] [1000 * 1]
```
<img src="https://github.com/user-attachments/assets/96e4eb65-128f-431e-b12f-2947d85f7a82" style="width: 40%; height: auto">  

위에 나열한 각 직사각형 크기는 모두 같은 크기다. 이를, 소수 판별법에 응용하면 각 앞의 숫자인 1 ~ 25는 N = 1000의 약수 m이면서 모두 대응되는 몫 q보다 작거나 같다. 그리고 √N = 31이므로 31보다 같거나 작은 약수 m은 [25 * 40]의 25이다. 즉, 제곱근보다 작거나 같은 25까지만 계산하면 소수인지 판별할 수 있다.  

**왜 제곱근까지만 확인하는 걸까?**  
<img src="https://github.com/user-attachments/assets/fe903050-dc23-431f-86d6-ce03c7851290" style="width: 40%; height: auto">  

N = 9, √N = 3이라고 가정할 때, 9는 3(m)과 3(q)으로 나눠지고, 두 약수인 3은 √N이면서 √N보다 작거나 같거나 크다. 그리고 3(m)과 3(q)는 3(q)과 3(m)과 같은 크기고 이미 먼저 확인했기 때문에 불필요한 계산이 된다. 따라서 제곱근 이하의 약수까지만 확인하면 된다.

### 코드
```Java
static boolean isPrimeSqrt(int n) {
         // Numbers less than or equal to 1 are not prime
        if (n <= 1)
            return false;

        // Check divisibility from 2 to the square root of n
        for (int i = 2; i <= Math.sqrt(n); i++)
            if (n % i == 0) 
                return false;

        // If no divisors were found, n is prime
        return true;
    }
```

### 시간복잡도
O(sqrt(N))

## 최적화된 √(N)(제곱근) 방법
### 코드
```Java
static boolean isPrimeOptimizedSqrt(int n) {
        // Check if n is 1 or 0
        if (n <= 1)
            return false;

        // Check if n is 2 or 3
        if (n == 2 || n == 3)
            return true;

        // Check whether n is divisible by 2 or 3
        if (n % 2 == 0 || n % 3 == 0)
            return false;
        
        // Check from 5 to square root of n
        // Iterate i by (i+6)
        for (int i = 5; i <= Math.sqrt(n); i = i + 6)
            if (n % i == 0 || n % (i + 2) == 0)
                return false;

        return true;
    }
```

### 시간복잡도
O(sqrt(N))  
<br>

# 소수 생성 체 알고리즘
소수 생성 알고리즘에는 에라토스테네스의 체, 세그먼트 체, 순다람의 체, 비트 단위 체 등이 있다. 그중에서 일반적으로 사용되는 알고리즘은 에라토스테네스의 체이다. 따라서 에라토스테네스의 체 알고리즘만을 다루어보려고 한다.

## 에라토스테네스의 체(Sieve of Eratosthenes)
![Sieve_of_Eratosthenes_animation](https://github.com/user-attachments/assets/d32b3741-3560-41f4-a034-c7460d64cbf1)  
출처: [워키백과 - 에라토스테네스의 체](https://ko.wikipedia.org/wiki/%EC%97%90%EB%9D%BC%ED%86%A0%EC%8A%A4%ED%85%8C%EB%84%A4%EC%8A%A4%EC%9D%98_%EC%B2%B4)  

주어진 수 N까지의 모든 소수를 빠르게 찾을 수 있는 알고리즘이다. 이 방법은 N이 1000만 이하일 때, 가장 효율적으로 소수를 구할 수 있다.

### 알고리즘 설명
1. **수 목록 만들기**  
1부터 N까지의 모든 수를 나열한다. 이때, 1은 소수가 아니므로 제외한다.  
2. **소수의 배수 지우기**  
첫 번째 소수인 2부터 시작하여, 그 배수들을 모두 지운다. 그 다음 소수인 3을 선택하여, 3의 배수들을 지운다.  
이 과정은 N까지 반복한다.  
3. **남은 수가 소수**  
모든 배수들을 지운 후 남은 수들은 소수이다.  

**단계별 실행 예시**  
```
1. N = 30이라면, 처음에는 1부터 30까지 모든 수를 나열한다.  
2. 2부터 시작하여 2의 배수(4, 6, 8, ...)를 모두 지운다. 그 다음 소수인 3을 선택하여, 3의 배수(9, 12, 15, ...)를 지운다. 
다음 소수인 5를 선택하고, 5의 배수(25, 30 등)를 지운다.  
3. 마지막으로 남은 수들인 2, 3, 5, 7, 11, 13, 17, 19, 23, 29가 소수가 된다.  
```

### 코드
```Java
static void sieveOfEratosthenes(int n) {
        // Create a boolean array "prime[0..n]" and
        // initialize all entries it as true. A value in
        // prime[i] will finally be false if i is Not a
        // prime, else true.
        boolean prime[] = new boolean[n + 1];
        for (int i = 0; i <= n; i++)
            prime[i] = true;

        for (int p = 2; p * p <= n; p++) {
            // If prime[p] is not changed, then it is a
            // prime
            if (prime[p] == true) {
                // Update all multiples of p greater than or
                // equal to the square of it numbers which
                // are multiple of p and are less than p^2
                // are already been marked.
                for (int i = p * p; i <= n; i += p)
                    prime[i] = false;
            }
        }

        // Print all prime numbers
        for (int i = 2; i <= n; i++) {
            if (prime[i] == true)
                System.out.print(i + " ");
        }
    }
```

### 시간복잡도
O(N*log(log(N)))  
<br>

# 참고
* [Prime Numbers](https://www.geeksforgeeks.org/prime-numbers)
* [Check for Prime Number](https://www.geeksforgeeks.org/check-for-prime-number/#efficient-approach-1-trial-division-method)
* [Sieve of Eratosthenes](https://www.geeksforgeeks.org/sieve-of-eratosthenes/)
* [03. 소수 구하기](https://wikidocs.net/205457)
* [Sieve of Eratosthenes](https://www.geeksforgeeks.org/sieve-of-eratosthenes)