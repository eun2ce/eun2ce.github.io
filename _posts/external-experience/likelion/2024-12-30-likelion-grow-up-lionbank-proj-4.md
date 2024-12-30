---
title: "[ 멋쟁이사자처럼 백엔드 스쿨 ] 은행 시스템 개선: stream 과 enum 을 활용한 메뉴 조회 기능 리팩토링"
date: 2024-12-30 11:47:00 +0900
categories: [ "external-experience", "likelion" ]
tags: [ "bootcamp", "java", "멋쟁이사자처럼", "부트캠프", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/external-experience/likelion/2024-12-27-likelion-grow-up-lionbank-proj-4/2024-12-30-11-48-30.png
  alt: "은행 시스템 개선: stream 과 enum 을 활용한 메뉴 조회 기능 리팩토링"
---

> 학습을 목표로 자바 프로젝트를 진행하고 있습니다.
> {: .prompt-info}

`Java`로 개발하다 보면 데이터를 필터링하거나 정렬하는 작업은 빈번하게 발생합니다.  
특히 반복문(forEach)을 사용하는 경우가 많지만, 더 선언적이고 간결한 방식으로 이를 처리할 방법(`Stream API`와 `Enum`)을 메뉴 조회 기능에 적용시키고 이를
통한 이점을 확인합니다.

## 비교

### forEach

```java
public static Menu findNameByValue(int value) {
  Menu menu = null;
  for (Menu m : values()) {
    if (m.value == value) {
      menu = m;
    }
  }
  if (menu == null) {
    throw new IllegalAccessError("Invalid menu value: " + value);
  }
  return menu;
}
```

### Stream

* 이점
  * 코드 간결성
  * 선언적 프로그래밍: 필터 조건과 결과 값을 명시적으로 나타내어 **의도**를 명확하게 표현
  * 안정성: `orElseThrow`를 통한 명시적 예외처리 ( forEach 의 경우 별도로 예외처리를 하지 않으면 `null exception`이 발생할 수 있음)

```java

import java.util.Arrays;

public static Menu findNameByValue(int value) {
  return Arrays.stream(values()).filter(menu -> menu.value == value).findFirst()
    .orElseThrow(IllegalAccessError::new);
}
```

## 결과

코드 변경 시 성능 차이가 크지 않다면, 가독성과 유지보수성을 우선적으로 고려하자.

> 자세한 패치
> 내용은 [이 곳](https://github.com/eun2ce/likelion/commit/bba1206be98473906e3d0f3b562be87ce3fa4919)을
> 참고해주세요.
> {: .prompt-info}
