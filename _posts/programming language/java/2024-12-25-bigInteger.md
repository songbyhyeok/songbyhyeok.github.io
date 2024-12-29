---
title: BigInteger
categories: java
---

# BigInteger?  
**배열 형태로 구성되어 있는 객체 타입으로서, 원시 타입의 범위로 표현할 수 없는 값들을 처리**  

## 원시타입과 범위 비교
* **4byte**  
    크기가 표현할 수 있는 범위는 -2,147,483,648 ~ 2,147,483,647 (대략20억, 10자리)  
* **8byte**  
    -9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807 (대략 900경, 19자리 -> 부호 없는 경우 1800경, 20자리) 
* **BigInteger**  
    값의 범위는 무한(Infinity), 대략 70Byte를 표현할 수 있다.

## 연산
```
import java.math.BigInteger;

public class BigIntegerArithmetic {
    public static void main(String[] args) {
        // BigInteger 객체 생성
        BigInteger num1 = new BigInteger("1234567890123456789012345678901234567890");
        BigInteger num2 = new BigInteger("987654321098765432109876543210987654321");

        // 덧셈
        BigInteger sum = num1.add(num2);
        System.out.println("덧셈: " + sum);

        // 뺄셈
        BigInteger difference = num1.subtract(num2);
        System.out.println("뺄셈: " + difference);

        // 곱셈
        BigInteger product = num1.multiply(num2);
        System.out.println("곱셈: " + product);

        // 나눗셈
        BigInteger quotient = num1.divide(num2);
        System.out.println("나눗셈(몫): " + quotient);

        // 나머지
        BigInteger remainder = num1.remainder(num2);
        System.out.println("나눗셈(나머지): " + remainder);
    }
}
```
```
// 결과
덧셈: 2222222221111111111011111111011111111111
뺄셈: 246913577913580246792456790679135802468
곱셈: 121932631112635269437510282091171268327999926428789906393981396679699740051
나눗셈(몫): 1
나눗셈(나머지): 246913577913580246792456790679135802468
```

## 형변환
```
import java.math.BigInteger;

public class BigIntegerConversion {
    public static void main(String[] args) {
        // BigInteger 객체 생성
        BigInteger bigInt = new BigInteger("12345");

        // BigInteger -> int 변환
        int intValue = bigInt.intValue(); // int 범위 내의 값만 변환
        System.out.println("int value: " + intValue);

        // BigInteger -> long 변환
        long longValue = bigInt.longValue();
        System.out.println("long value: " + longValue);

        // BigInteger -> double 변환
        double doubleValue = bigInt.doubleValue();
        System.out.println("double value: " + doubleValue);

        // BigInteger -> String 변환
        String stringValue = bigInt.toString();
        System.out.println("String value: " + stringValue);

        // BigInteger -> byte[] 변환
        byte[] byteArray = bigInt.toByteArray();
        System.out.println("byte[] value: " + java.util.Arrays.toString(byteArray));
    }
}
```
```
/// 결과
int value: 12345
long value: 12345
double value: 12345.0
String value: 12345
byte[] value: [0, 0, 0, 0, 0, 0, 48, 57]
```

## 비교
```
import java.math.BigInteger;

public class BigIntegerComparison {
    public static void main(String[] args) {
        // BigInteger 객체 생성
        BigInteger bigInt1 = new BigInteger("12345");
        BigInteger bigInt2 = new BigInteger("67890");
        BigInteger bigInt3 = new BigInteger("12345");

        // BigInteger 비교: bigInt1 vs bigInt2
        int result1 = bigInt1.compareTo(bigInt2);
        if (result1 < 0) {
            System.out.println("bigInt1 is less than bigInt2");
        } else if (result1 > 0) {
            System.out.println("bigInt1 is greater than bigInt2");
        } else {
            System.out.println("bigInt1 is equal to bigInt2");
        }

        // BigInteger 비교: bigInt1 vs bigInt3
        int result2 = bigInt1.compareTo(bigInt3);
        if (result2 < 0) {
            System.out.println("bigInt1 is less than bigInt3");
        } else if (result2 > 0) {
            System.out.println("bigInt1 is greater than bigInt3");
        } else {
            System.out.println("bigInt1 is equal to bigInt3");
        }
    }
}
```
```
// 결과
bigInt1 is less than bigInt2
bigInt1 is equal to bigInt3
```

## 언제 유용한가?
BigInteger는 정수의 크기 제한이 필요 없고, 정확한 계산이 필요한 경우에 유용하다. 개인적으로 금융, 특정 알고리즘에서 필요함을 느꼈다.  
<br>

# 참고
* [Java에서 큰 수 다루기 (BigInteger)](https://lsmman.tistory.com/47)



