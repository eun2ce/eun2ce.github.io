---
title: "[멋쟁이사자처럼 백엔드 스쿨] Java 프로그래밍 기초 - static, final, static final 차이"
description: "static, final, static final 이 어떻게 다른지 정리합니다."
date: 2024-12-06 16:30:00 +0900
categories: [ "etc", "project" ]
tags: [ "bootcamp", "static", "java", "javac", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2024-12-06-likelion-java-static-final-static-final-2024-12-06-11-48-18.webp
  alt: "[ 멋쟁이사자처럼 백엔드 스쿨 ] Java 프로그래밍 기초 - static, final, static final 차이"
---

수업한 내용들을 기록하지만 개인적으로 요약정리 한 내용들이 포함되어있을 수 있습니다.

## Summary

| **항목**         | **`static`**    | **`final`**    | **`static final`**        |
|----------------|-----------------|----------------|---------------------------|
| **변수**         | 모든 객체가 공유       | 선언 후 값 변경 불가   | 전역 상수처럼 사용 가능             |
| **메서드**        | 인스턴스 생성 없이 호출 가능 | 재정의 불가         | 인스턴스 생성 없이 호출 가능하며 재정의 불가 |
| **클래스**        |                 | 상속 불가          |                           |
| **메모리상 저장 위치** | 클래스 영역(메서드 영역)  | 인스턴스 또는 클래스 영역 | 클래스 영역(메서드 영역)            |

* 클래스 선언 자체에 `static`을 붙일 수 없음
* `static final`는 변수(상수) 정의 시 사용

---

표만보면 이해하기 어려우니 간단한 예시와 함께 확인해보자.

## static

* 모든 객체가 공유
* 인스턴스 생성 없이 호출 가능
* 메모리 상 클래스 영역(메서드 영역)에 저장

```java
class StaticExample {
  static int count = 0; // static 변수

  static void increment() { // static 메서드
    count++;
  }
}

public class Main {
  public static void main(String[] args) {
    StaticExample.increment(); // 클래스명으로 접근
    System.out.println(StaticExample.count); // 출력: 1
  }
}
```

## final

* 값 변경, 재 정의, 상속 불가
* 초기화 후 불변성 보장

```java
class FinalExample {
  final int MAX_VALUE = 100; // final 변수

  final void display() { // final 메서드
    System.out.println("This cannot be overridden");
  }
}

// final 클래스는 상속 불가
final class FinalClass {
}

class ExampleClass extends FinalClass {

}

// final 메서드는 재정의 불가
// class SubClass extends FinalExample {
//     @Override
//     void display() {
//         System.out.println("Override attempt");
//     }
// }
```

![final class 상속 시 생기는 문제](/assets/img/posts/2024-12-06-likelion-java-static-final-static-final-2024-12-06-11-48-18.webp)

### static final

* 변경 불가능한 **상수**

```java
class StaticFinalExample {
  static final double PI = 3.141592; // static final 변수

  static final int getDoublePI() { // static final 메서드
    return (int) (PI * 2);
  }
}

public class Main {
  public static void main(String[] args) {
    System.out.println(StaticFinalExample.PI); // 출력: 3.141592
    System.out.println(StaticFinalExample.getDoublePI()); // 출력: 6
  }
}
```
