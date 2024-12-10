---
title: "Java String 과 String Constant Pool"
date: 2024-12-10 16:03:00 +0900
categories: [ "language", "java" ]
tags: [ "constant pool", "string", "java", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/language/java/2024-12-10-java-string-constant-pool/2024-12-10-16-14-11.png
  alt: "Java String 과 String Constant Pool"
---

Java 에서 **String Pool**은 문자열을 효율적으로 관리하기 위한 메모리 영역입니다.  
이는 **String Interning**이라고도 불리며, Java 프로그램이 메모리를 절약하고 성능을 향상시킬 수 있도록 도와줍니다.

### String Pool 의 개념

Java 에서 문자열은 불변(immutable) 객체입니다.  
String Pool 은 문자열 리터럴을 저장하는 특별한 메모리 공간으로, 프로그램이 같은 문자열 값을 반복해서 생성할 때 메모리 사용을 줄이기 위해 사용됩니다.  
즉, 같은 문자열 리터럴은 **String Pool**에 단 하나만 존재하고, 이를 참조하는 변수는 같은 객체를 참조하게 됩니다.

예를 들어, 코드에서 다음과 같은 문자열을 선언할 때:

```java
String str1 = "Hello";
String str2 = "Hello";
```

`str1`과 `str2`는 **같은 메모리 주소를 참조**하게 됩니다. (`String Pool`에 `"Hello"`라는 문자열 리터럴이 이미 존재하기 때문)  
이와 같은 방식으로 메모리 사용을 줄일 수 있습니다.

### String Pool 의 동작 원리

- **리터럴 문자열**: 코드에 직접 작성된 문자열(예: `"Hello"`)은 컴파일 시 `String Pool`에 저장됩니다.
- **new 연산자**: `new` 키워드를 사용해 문자열을 생성할 경우, `String Pool`에 문자열이 자동으로 추가되지 않고, **새로운 `String` 객체**가 힙 메모리에 생성됩니다.

```java
String str3 = new String("Hello");
System.out.println(str1 == str3); // false, 서로 다른 객체를 참조
```

### String Pool 의 장점

- **메모리 절약**: 중복된 문자열이 메모리 내에 여러 번 저장되지 않아 메모리 사용량을 줄입니다.
- **성능 향상**: 문자열 비교 시 `==` 연산자를 사용하여 참조 비교가 가능해져 성능이 더 빠를 수 있습니다.

### String Pool 을 활용한 예시

```java
String a = "World";
String b = "World";
System.out.println(a == b); // true, 같은 객체를 참조

String c = new String("World");
System.out.println(a == c); // false, 서로 다른 객체를 참조
```

### String Intern 메서드

`String` 클래스의 `intern()` 메서드를 사용하면 `String Pool`에 문자열을 추가할 수 있습니다.  
만약 같은 문자열이 `String Pool`에 이미 존재하면 해당 객체의 참조를 반환하고, 없으면 새로 추가합니다.

```java
String d = new String("Java").intern();
String e = "Java";
System.out.println(d == e); // true, 같은 객체를 참조
```

![inter method](/assets/img/posts/language/java/2024-12-10-java-string-constant-pool/2024-12-10-16-14-11.png)


이와 같은 방식으로 `String Pool`은 Java 의 문자열 처리에 있어 중요한 역할을 하며, 메모리와 성능을 최적화하는 데 도움을 줍니다.
