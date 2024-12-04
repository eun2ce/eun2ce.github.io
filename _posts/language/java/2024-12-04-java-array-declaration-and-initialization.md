---
title: "Java 배열 선언과 초기화"
pin: false
math: false
mermaid: false
categories: [ "java", "language" ]
tags: [ "java", "프로그래밍" ]
date: 2024-12-04 13:00:00 +0900
image:
  path: /assets/img/posts/language/java/2024-12-04-java-array-declaration-and-initialization/20241204-14-01-00.png
  alt: "Java 배열 선언과 초기화"
---

## Java 배열 선언과 초기화: `int[] arr1 = {};` vs `int[] arr2;`

java 에서 배열 선언과 초기화는 코드의 명확성과 사용 용도에 따라 적절히 구분되어야 합니다.  
`int[] arr1 = {};`와 `int[] arr2;`로 예를들어 설명합니다.

### int[] arr1 = {}

**초기화된 빈 배열**

- **의미**
  - 크기가 0인 배열을 생성
  - 배열은 생성과 동시에 메모리 할당
  - 기본값으로 초기화

- **특징**:
  - `arr1`은 **`null`이 아닌 배열 객체**를 참조
  - 배열의 크기를 변경할 수 없음 (데이터를 추가하려면 새로운 배열을 생성)

- **사용 예시**:

```java
public class ArrayExample1 {
  public static void main(String[] args) {
    int[] arr1 = {}; // 크기가 0인 배열
    System.out.println("Array length: " + arr1.length); // 출력: 0
  }
}
```

- **장점**:
  - NPE(NullPointerException) 방지 (초기화하면서 빈 배열을 명시적으로 생성하기때문)

### int[] arr2;

**배열 참조 변수 선언**

- **의미**: 배열을 참조할 수 있는 변수를 선언(실제 배열 객체는 생성 **x**)
- **특징**:
  - `arr2`는 **초기화되지 않은 상태** (사용 시 컴파일 오류)
  - 명시적으로 배열 객체를 생성 또는 다른 배열을 참조
- **사용 예시**:

```java
public class ArrayExample2 {
  public static void main(String[] args) {
    int[] arr2; // 선언만 함
    arr2 = new int[5]; // 배열 객체 생성
    System.out.println("Array length: " + arr2.length); // 출력: 5
  }
}
```

- **주의**:  
  배열을 초기화하지 않고 접근하면 **컴파일 오류**가 발생

```java
public class ArrayExample3 {
  public static void main(String[] args) {
    int[] arr2;
    // System.out.println(arr2.length); // 컴파일 오류: 초기화되지 않은 변수 사용
  }
}
```

### 주요 차이점

| 구분                       | `int[] arr1 = {};` | `int[] arr2;`      |
|--------------------------|--------------------|--------------------|
| **초기화 여부**               | 크기 0으로 초기화된 배열     | 초기화되지 않음           |
| **메모리 할당**               | 즉시 할당              | 명시적으로 할당 필요        |
| **NullPointerException** | 발생하지 않음            | 초기화 없이 사용 시 발생 가능  |
| **사용 용도**                | 빈 배열을 기본값으로 사용     | 이후 동적으로 배열을 설정할 경우 |

### 고려사항

- **빈 배열 필요 시**: `int[] arr1 = {};`를 사용해 명시적으로 초기화
  - 예: 메서드 반환값이 빈 배열인 경우
- **유연한 초기화가 필요할 때**: `int[] arr2;`를 선언 후 조건에 따라 초기화
  - 예: 배열 크기나 데이터를 런타임에 결정하는 경우

### 결론

- **명시적으로 빈 배열이 필요하다면** `int[] arr1 = {};`를 사용
- **배열 크기나 데이터가 동적으로 결정된다면** `int[] arr2;`를 선언 후 조건에 따라 초기화
