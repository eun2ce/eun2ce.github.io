---
title: "[멋쟁이사자처럼 백엔드 스쿨] Java 프로그래밍 기초 - 오류(Error) 와 예외 (Exception) 의 차이"
date: 2024-12-11 15:45:00 +0900
categories: [ "external-experience", "likelion" ]
tags: [ "bootcamp", "error", "exception", "java", "멋쟁이사자처럼", "부트캠프", "예외처리", "프로그래밍" ]
pin: false
math: false
mermaid: false
---

오류와 예외 처리를 분류하기 어려운 상황이 있습니다.
이번 글에서는 오류와 예외의 차이를 정의하고 적합하게 사용하는 방법을 제시합니다.

## 상속 구조 

![throwable](/assets/img/posts/external-experience/likelion/2024-12-11-likelion-java-try-catch-finally/2024-12-11-16-24-30.png)

* 오류(Error): 수습하기 어려운 문제
* 예외(Exception): 개발자 또는 사용자의 실수로 발생한 문제

### 오류(Error)

수습하기 어려운 문제

* StackOverFlowError: 호출의 깊이, 재귀가 지속되어 stack overflow 발생
* OutOfMemoryError: JVM 이 할당한 메모리 부족
  * GC에 의해 추가 메모리 확보가 어려운 경우

### 예외(Exception)

* NullPointerException: 객체가 필요한 경우 null 을 사용하려 시도한 경우
* IllgelArgumentException: 메서드 허가 불가 또는 부적절한 argument 를 받은 경우

> 예외는 오류와 다르게 개발자가 임의로 예외 시킬 수 있음
{: .prompt-info }

특정 예외처리에 따라 이 구분이 절대적이지 않지만 일반적으로 사용하는 Exception 과 Error 의 사용 용도는 위와 같습니다.

번외로


### 멀티 exception 처리 하는 방법

긴 코드나 과하게 분산 된 코드는 번잡하게 보일 수 있고, 불필요한 자원을 낭비할 수 있습니다.  
여러가지 exception 을 처리하는 스킬에 대해 작성합니다.

```
try {
    // ...
} catch (NullPointException | ArrayIndexOutOfBoundsExcetion e) {
    if(e instanceOf NullPointException) {
    	// ...
    } else if(e instanceOf ArrayIndexOutOfBoundsExcetion) {
    	// ...
    }
}
```
