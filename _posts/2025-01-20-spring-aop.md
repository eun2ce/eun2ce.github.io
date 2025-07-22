---
title: "[spring] AOP"
description: ""
date: 2025-01-20 17:30:00 +0900
categories: [ "programming", "spring" ]
tags: [ "aop", "spring", "java", "programming" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-01-20-spring-aop-2025-01-20-20-09-36.png
  alt: "[Spring] AOP"
---

## AOP(Aspect Oriented Programming)란?

> AOP is a programming paradigm that aims to increase modularity by allowing the separation of
> cross-cutting concerns. It does this by adding additional behavior to existing code without
> modifying the code itself.

[https://www.baeldung.com](https://www.baeldung.com)에 따르면 AOP(Aspect-Oriented Programming)는 횡단 관심사를
분리하여 모듈화하고, 기존 코드에 영향을 주지 않으면서 추가 동작을 삽입하는 프로그래밍 방식입니다.

횡단 관심사란 로깅, 인증, 트랜잭션 관리와 같이 여러 모듈에서 공통적으로 필요한 기능들을 의미합니다.
이러한 관심사는 중복된 코드로 인해 가독성 저하, 유지보수 난이도 증가 등의 문제를 일으킵니다.

AOP는 이를 해결하기 위해 **핵심 비즈니스 로직과 공통 기능을 분리**하여, 코드 중복을 줄이고 유지보수성을 높입니다.

로깅을 사용하여 실행 중에 의미있는 정보를 기록하는 코드를 예로 들어봅니다.

```java
public String greet(String name) {
  logger.debug(">> greet() - {}", name);
  String result = String.format("Hello %s", name);
  logger.debug("<< greet() - {}", result);
  return result;
}
```

위와같이 로깅문은 코드에서 불필요한 잡동사니처럼 느껴질 수 있습니다.
게다가 코드의 복잡성이 증가했습니다. 로깅 없이 메서드를 작성하면 아래와 같습니다.

```java
public String greet(String name) {
  return String.format("Hello %s", name);
}
```

게다가 다른 메서드에 로깅기능을 추가하려면 중복코드가 발생하게 됩니다.

이때 사용하는 것이 AOP라고 할 수 있습니다.

AOP는 공통 관심 사항과 핵심 관심 사항을 분리해주는 기능을 가지고 있습니다.
여기서 공통 관심 사항은 **로깅**기능이고 핵심 관심 사항은 비즈니스 로직이 됩니다.

## AOP를 사용한 로깅

Spring에서 AOP를 구현하는 방법 중 한가지는 @Aspect주석이 달린 Spring 빈(bean)을 사용하는 것입니다.

```java

@Aspect
@Component // Spring은 @Aspect를 컴포넌트로 취급하지 않으므로 Spring에서 관리하고 컴포넌트 스캐닝을 통해 감지해야 하는 빈임을 나타내기 위해 추가
public class LoggingAspect {

}
```

### 로깅 포인트컷 정의

여기서는 `com.example.aop.logging`의 public 메서드만 포함하는 `pointcut` 표현식으로 정의했습니다.

```java
// pointcut
@Pointcut("execution(public * com.example.aop.logging.*.*(..))")
private void publicMethodsFromLoggingPackage() {
}
```

### Around Advice 사용

[around advice](https://www.baeldung.com/spring-aop-advice-tutorial#around)를 사용해 메서드 호출 전,후 로깅 기능을
추가합니다.

```java
// advice
@Around(value = "publicMethodsFromLoggingPackage()")
public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
  Object[] args = joinPoint.getArgs();
  String methodName = joinPoint.getSignature().getName();
  logger.info(">> {" + methodName + "}() - {" + Arrays.toString(args) + "}");
  Object result = joinPoint.proceed(); // 내가 작성한 함수 실행 여기서는 greetingService가 된다.
  logger.info("<< {" + methodName + "}() - {" + result + "}");
  return result;
}
```

이렇게 하여 핵심 관심 사항과 정보를 기록하는 공통 관심 사항을 분리했습니다.

로깅 로직을 별도의 공통 로직으로 만들었습니다.

작성한 코드는 아래와 같이 테스트해볼 수 있습니다.

```java
	@Test
	void greetingServiceTest() {
		String result = greetingService.greet("eun2ce");
		assertNotNull(result);
		assertEquals("Hello eun2ce", result);
	}
```

### 결과

![결과](/assets/img/posts/2025-01-20-spring-aop-2025-01-20-19-54-56.png)

## AOP 동작 방식

![동작방식](/assets/img/posts/2025-01-20-spring-aop-2025-01-20-20-09-36.png "출처: baeldung")

여러 `Join point`들이 실행되고, 이때 특정 `Pointcut`으로 필터링 된 포인트 컷이 호출되는 순간 advice 객체의 어드바이스 메서드가 실행됩니다.

어드바이스 메서드 시점은 개발자가 설정한대로 호출이되며, 포인트컷으로 지정한 메서드가 호출될 때 어드바이스 메서드를 삽입하도록 하는 설정을 `Aspect`라고 합니다.
이 `Aspect`설정에 따라 `Weaving(위빙)`이 처리가 됩니다.

> weaving: `Pointcut`에 의해서 결정된 타겟의 `Join Point`에 부가기능(Advice)를 삽입하는 과정

전체 코드 예제는 [GitHub](https://github.com/eun2ce/likelion/tree/main/aop)에서 확인할 수 있습니다.
