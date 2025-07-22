---
title: "[spring data jpa] No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call 해결"
description: ""
date: 2025-01-21 10:08:00 +0900
categories: [ "programming", "spring" ]
tags: [ "exception", "spring", "java", "jpa", "programming" ]
pin: false
math: false
mermaid: false
---

```
jakarta.persistence.TransactionRequiredException: No EntityManager with actual transaction available for current thread - cannot reliably process 'persist' call
```

"현재 스레드에 `EntityManager`가 없어 persist 할 수 없다."는 문제입니다.

그러나 로그를 제대로 살펴보면 <ins>actual transaction available for current thread</ins>라고 나와있습니다.

> 즉, 내가 하는 작업에 트랜잭션 선언이 되어있지 않다. 정도..

`@Transactional`을 빼먹었습니다.

기본적으로 JPA는 transaction을 기반으로 작동하게 되어있습니다.

transaction단위에 따라 1차 캐시영역에 있는 객체들이 DB에 flush되어 영속화하기 때문입니다.

하지만 그런 영속작업을 하는 persist() 메서드에 객체가 들어갔으나 가능한 transaction이 존재하지 않았기에 해당 에러가 나타난 것 입니다.

> 서비스 혹은 클래스에 미리 @Transactional을 선언하고  
> 클래스에는 @Transactional(readOnly = true) / 메서드에는 @Transactional 을 붙여  
> read, write 트랜잭션을 구분하는 것도 잊지 말자.
