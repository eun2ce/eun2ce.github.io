---
title: "Java Class 형변환"
description: "형변환(type conversion)은 캐스팅(casting) 이라고도 불립니다. java 에서의 class 형 변환에 대해 알아봅니다."
date: 2024-12-09 17:02:00 +0900
categories: [ "language", "java" ]
tags: [ "casting", "class", "java", "프로그래밍", "클래스" ]
pin: false
math: false
mermaid: false
---

## Summary

* 업캐스팅: 부모 타입으로 변환(자동,암묵적)
* 다운캐스팅: 자식 타입으로 변환(수동, 명시적)
* 형변환 전에는 **instanceof**를 통해 안정성을 확인하는 것이 좋다.
  * 객체가 특정 클래스인지 확인

```java
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Dog();

        if (animal instanceof Dog) {
            System.out.println("This is a Dog");
        } else if (animal instanceof Cat) {
            System.out.println("This is a Cat");
        } else {
            System.out.println("Unknown Animal");
        }
    }
}
```

## 사전 조건

* 서로 상속 관계
* 부모 클래스와 자식 클래스 간에만 형변환 가능

## **암묵적 형변환**

부모(Parent) 객체가 자식(Child) 객체의 상위 클래스인 경우

```java
Parent p = new Child();
```

## **명시적 형변환**

```java
Parent p = new Child();
Child c = (Child) p;
```
