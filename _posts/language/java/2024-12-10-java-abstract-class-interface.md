---
title: "Java 추상 클래스와 인터페이스 차이"
date: 2024-12-10 15:05:00 +0900
categories: [ "language", "java" ]
tags: [ "abstract class", "interface", "java", "인터페이스", "추상클래스" ]
pin: false
math: false
mermaid: false
---

일반적으로 아래와 같은 선택의 기준을 통해 우리는 추상 클래스와 인터페이스 사용 시점에 대해 구분합니다.
이 내용을 바탕으로 궁금한 내용이 생겼던 것을 찾아보고 정리하여 공유합니다.

## 선택 기준

* 추상 클래스: 상속, 확장하여 사용하기 위한 것
* 인터페이스: 동일한 사용방법과 동작을 보장하기 위함

즉, 추상클래스는 **공용 메서드**가 존재하나 `abstract`메서드를 사용하여 구현을 강제하고 인터페이스는 **공용 메서드**가 없으므로 반드시 **메서드를 구현**해야합니다.

그런데 java8 부터 인터페이스에 **default**메서드가 추가 되었고 `default`를 선언하면, 인터페이스를 상속받는 클래스에서 구현을 하지 않고도 사용할 수 있게 되었습니다.

예제

```java
interface Animal {

  final int age = 1;

  void name();

  default int getAge() {
    return age;
  }
}

class Dog implements Animal {

  @Override
  public void name() {
    return "cookie";
  }
}

public class Main {

  public static void main(String[] args) {
    Animal animal;
    animal = new Dog();
    System.out.println(animal.getAge());
  }
}
```

그러면 `default`메서드가 추상 클래스의 일반 메서드와 동일하게 작동하는데 이 때는 어떻게 구별하는게 좋을까

## 결론

`default` 메서드가 인터페이스에 추가된 것은 **기존 구현체에서 오류를 방지하기 위한 하나의 해결책**입니다. 그러나 이를 너무 확대해석하거나 남용하지 않도록 주의해야 합니다. 추상 클래스와 인터페이스는 본래의 목적에 맞게 사용해야 하며, 그 존재 이유를 다시 한 번 되새기는 것이 중요합니다.

### 핵심 정리

* **새로운 문법을 통해 문제를 해결할 수 있는 길이 하나 열렸다**는 정도로 생각하자.
* 인터페이스의 본질은 변하지 않으며, 설계의 계약을 정의하는 목적은 여전히 유효하다.
* 추상 클래스와 인터페이스의 존재 이유를 명확히 하고, 그에 맞게 적절히 활용하자.
