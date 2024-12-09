---
title: "[ 멋쟁이사자처럼 백엔드 스쿨 ] Java 프로그래밍 기초 - overloading vs overriding"
description: "static, final, static final 이 어떻게 다른지 정리한다."
date: 2024-12-02 11:28:00 +0900
categories: [ "external-experience", "likelion" ]
tags: [ "bootcamp", "overloading", "overriding", "java", "javac", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
---

수업한 내용들을 기록하지만 개인적으로 요약정리 한 내용들이 포함되어있을 수 있습니다.

## Summary

* 오버로딩(overloading): 기존에 없는 새로운 메서드를 **정의**
* 오버라이딩(overriding): 상속받은 메서드의 내용을 **변경**

| **구분**       | **`오버로딩`** | **`오버라이딩`** |
|--------------|------------|-------------|
| **메서드 이름**   | 동일         | 동일          |
| **매게변수, 타입** | 다름         | 동일          |
| **리턴타입**     | 상관없음       | 동일          |

---

간단한 예시와 함께 확인해보자.

## overloading

```java
class Calculator {

  // 정수 덧셈
  int add(int a, int b) {
    return a + b;
  }

  // 실수 덧셈
  double add(double a, double b) {
    return a + b;
  }

  // 세 숫자 덧셈
  int add(int a, int b, int c) {
    return a + b + c;
  }
}

public class Main {

  public static void main(String[] args) {
    Calculator calc = new Calculator();

    System.out.println(calc.add(3, 5));           // 정수 덧셈 호출
    System.out.println(calc.add(3.5, 5.5));       // 실수 덧셈 호출
    System.out.println(calc.add(1, 2, 3));        // 세 숫자 덧셈 호출
  }
}
```

## overriding

```java
class Parent {

  // 부모 클래스의 메서드
  void greet() {
    System.out.println("Hello from Parent!");
  }
}

class Child extends Parent {

  // 메서드 오버라이딩
  @Override
  void greet() {
    System.out.println("Hello from Child!");
  }
}

public class Main {

  public static void main(String[] args) {
    Parent parent = new Parent();
    Parent child = new Child();

    parent.greet(); // Parent 의 greet 호출
    child.greet();  // Child 의 greet 호출
  }
}
```

### `@overriding` 어노테이션 사용 권장

* 장점
  * 컴파일 시점에 오류 방지 가능
  * 가독성 향상

#### 메서드에 사용하는 경우

```java
class Animal {

  // 동물이 소리를 내는 메서드
  void sound() {
    System.out.println("Some generic animal sound");
  }
}

class Dog extends Animal {

  @Override
  void sound() {
    // 부모 메서드를 오버라이딩
    System.out.println("Woof Woof!");
  }
}

public class Main {

  public static void main(String[] args) {
    Animal dog = new Dog();
    Animal cat = new Cat();

    dog.sound(); // Dog 클래스의 sound 메서드 호출
  }
}
```

#### 인터페이스에 사용하는 경우

* 장점
  * 인터페이스에 선언 된 메서드가 제대로 구현 되었는지 컴파일 타임에 확인

* 주의사항
  * 접근 제한자는 반드시 `public`

```java
// 인터페이스 정의
interface Animal {

  void sound(); // 추상 메서드

  void move();  // 추상 메서드
}

// Dog 클래스가 인터페이스를 구현
class Dog implements Animal {

  @Override
  public void sound() {
    System.out.println("Woof Woof!");
  }

  @Override
  public void move() {
    System.out.println("The dog runs swiftly.");
  }
}

// Cat 클래스가 인터페이스를 구현
class Cat implements Animal {

  @Override
  void sound() {  // 컴파일 error, 접근 제한자가 public 이 아님
    System.out.println("Meow Meow!");
  }

  @Override
  public void move() {
    System.out.println("The cat jumps gracefully.");
  }
}

public class Main {

  public static void main(String[] args) {
    Animal dog = new Dog();
    Animal cat = new Cat();

    dog.sound(); // Dog 클래스의 sound 메서드 호출
    dog.move();  // Dog 클래스의 move 메서드 호출

    cat.sound(); // Cat 클래스의 sound 메서드 호출
    cat.move();  // Cat 클래스의 move 메서드 호출
  }
}
```

