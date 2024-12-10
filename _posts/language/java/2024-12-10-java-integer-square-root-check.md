---
title: "Java 두 원의 위치관계 비교"
date: 2024-12-10 11:50:00 +0900
categories: [ "language", "java" ]
tags: [ "1002", "백준", "알고리즘", "java" ]
pin: false
math: false
mermaid: false
---

두 원의 위치관계를 계산할 때, 좌표간의 거리 `((𝑥₂- 𝑥₁)²+(𝑦₂-𝑦₁)²)½`를 구하기 위해 아래와 같은 수식을 사용하는 경우가 많다.

```java
double distance = Math.sqrt(Math.pow(x2 - x1, 2) + Math.pow(y2 - y1, 2)); 
```

그러나 실제로 좌표를 비교할 때 `==` 연산자를 사용하게 되는데 `double`, `float`는 오차가 발생할 수 있다.

> 백준은 왜인지 모르겠는데 풀리지만 절대 권장하는 방법은 아니다.
> {: .prompt-danger }

이유는 부동소수점 타입이 **근사치**로 처리되기 때문

> 이를 부동소수점 연산의 정확도 문제(Floating-point precision issue) 라고 한다.

간단한 예로

```java
public class Main {

  public static void main(String[] args) {
    double a = 0.1;
    double b = 0.2;
    double c = 0.3;

    if (a + b == c) {
      System.out.println("true");
    } else {
      System.out.println("false");
    }
  }
}

// 실행결과
// false
```

## 그래서

거리를 구할 때 `Math.sqrt()`를 사용하는게 아니라 제곱이 되어있는 형태,  `(𝑥₂-𝑥₁)²+(𝑦₂-𝑦₁)²`를 이용하자.

e.g) 각 원의 반지름이 r1, r2 라고 했을 때 비교하는 방법

```java
private static int func(int x1, int y1, int r1, int x2, int y2, int r2) {
  int distance = (int) ((Math.pow(x2 - x1, 2)) + (Math.pow(y2 - y1, 2)));

  if (x1 == x2 && y1 == y2 && r1 == r2) { // 동접원
    return -1;
  } else if (distance_pow > Math.pow(r2 + r1, 2)) { // 동떨어져있을 때
    return 0;
  } else if (distance_pow < Math.pow(r2 - r1, 2)) { // 한 원이 포함될 때
    return 0;
  } else if (distance_pow == Math.pow(r2 + r1, 2)) { // 외접
    return 1;
  } else if (distance_pow == Math.pow(r2 - r1, 2)) { // 내접
    return 1;
  } else {
    return 2;
  }
}
```

