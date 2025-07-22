---
title: "[java] Optional을 사용하는 이유와 바른 사용법 (feat. 안티패턴, NPE 피하기)"
date: 2025-01-20 16:00:00 +0900
categories: [ "language", "java" ]
tags: [ "npe", "nullpointexception", "optional", "java", "사용법", "프로그래밍" ]
pin: false
math: false
mermaid: false
---

프로그래밍을 하다보면 `null`처리를 필수적으로 하게됩니다.
기존에는 `null`체크를 해서 분기처리(`if`문같은)하는 형태였다면 자바 8 이후부터는 `optional(옵셔널)`을 사용합니다.

Optional을 공부하며 사용해야하는 이유에 대해 알아봅니다.

> optional 의 등장 배경
> 기존에 런타임 시 NPE(NullPointException) 발생 > NPE방어 하기 위해 추가한 null check 추가 시 가독성 떨어짐
> 이를 개선하기 위해 java 8에 `java.util.Optional<T>`라는 새로운 클래스가 추가

## Java Optional이란?

"존재할 수도 있지만 안 할수도 있는 객체"

* `null`이 될 수도 있는 객체를 감싸고 있는 일종의 `wrapper`클래스
* 기존엔 객체를 직접 `null`인지 체크했다면, `optional`의 등장으로 `null`다루기가 쉬워짐

## 사용목적

메서드의 리턴타입으로 **결과 없음**을 명확하게 표현하기 위한 용도로 사용됩니다.

java 8 이전에는 Optional이라는 클래스가 없어 메서드의 리턴 타입으로 '결과 없음'을 표기하는데에 `null`을 사용했습니다.

그러나, 객체 값이 null인 경우 null체크 없이 해당 객체의 메서드를 호출하는 등, 부주의하게 사용되어 NPE(NullPointException)로 프로그램이 죽는 상황이
발생하게 되었고
이런 상황을 방지하고 안전한 프로그래밍을 위해 `optional`이 등장하게 됩니다.

> 추가로,
> null 값을 반환해야 하는 경우에 Optional로 감싼 객체로 empty 값을 표현하여 결괏값을 넘겨주면, null로 인한 오류 문제를 없앨 수 있습니다.

Optional은 **리턴 타입의 용도로 제한적으로 사용**되도록 설계되었습니다. 

> API Note:
> Optional is primarily intended for use as a method return type where there is a clear need to represent “no result,” and where using null is likely to cause errors. A variable whose type is Optional should never itself be null; it should always point to an Optional instance.  
> 메서드가 반환할 결과값이 ‘없음’을 명백하게 표현할 필요가 있고, null을 반환하면 에러를 유발할 가능성이 높은 상황에서 메서드의 반환 타입으로 Optional을 사용하자는 것이 Optional을 만든 주된 목적이다.  
> Optional 타입의 변수의 값은 절대 null이어서는 안 되며, 항상 Optional 인스턴스를 가리켜야 한다.

> 자세한 내용은 [이 곳](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html)을 참고하세요.
{: .prompt-info}

이를 통해:

1. 빈 결과를 명확히 표현할 수 있고,
2. null을 반환하거나 예외를 던지는 방식보다 안전하며,
3. 더 직관적인 코드를 작성할 수 있습니다.


## 문법

Optional 사용 시 잘못 사용하는 안티패턴과 올바른 사용을 자바 8 기준으로 갈무리합니다.

### isPresent()-get() 대신 orElse()/orElseGet()/orElseThrow()

안티패턴: isPresent()로 체크 후 get() 호출.
권장: orElse()나 orElseThrow() 사용으로 코드 간결화.

```java
public class Test {

  public static void main(String[] args) {
// 안 좋음
    Optional<User> user = ...;
    if (user.isPresent()) {
      return user.get();
    } else {
      return null;
    }

// 좋음
    Optional<User> user = ...;
    return user.orElse(null);

// 안 좋음
    Optional<User> user = ...;
    if (user.isPresent()) {
      return user.get();
    } else {
      throw new NoSuchElementException();
    }

// 좋음
    Optional<User> user = ...;
    return user.orElseThrow(() -> new NoSuchElementException());
  }
}
```

### orElse(new ...) 대신 orElseGet(() -> new ...)

문제: orElse()는 Optional 값 여부와 상관없이 항상 실행됨.
권장: 값이 없을 때만 실행되는 orElseGet() 사용.

```java
public class Test {

  public static void main(String[] args) {
// 안 좋음
    Optional<Member> member = ...;
    return member.orElse(new Member());  // member에 값이 있든 없든 new Member()는 무조건 실행됨

// 좋음
    Optional<Member> member = ...;
    return member.orElseGet(Member::new);  // member에 값이 없을 때만 new Member()가 실행됨

// 좋음
    Member EMPTY_MEMBER = new Member();
    // ... 중략
    Optional<Member> member = ...;
    return member.orElse(EMPTY_MEMBER);  // 이미 생성됐거나 계산된 값은 orElse()를 사용해도 무방
  }
}
```

### 단순 값 조회는 Optional 대신 null 비교

Optional은 무겁기 때문에 단순 값을 반환할 땐 null 체크 사용.

```
// 안 좋음
return Optional.ofNullable(status).orElse(READY);

// 좋음
return status != null ? status : READY;
```

### 컬렉션은 Optional대신 비어있는 컬렉션 반환

컬렉션은 Collections.emptyList() 등으로 비어있는 상태를 반환.

> 마찬가지 이유로 Spring Data JPA Repository 메서드 선언 시 다음과 같이 컬렉션을 Optional로 감싸서 반환하는 것은 좋지 않다.
> 컬렉션을 반환하는 Spring Data JPA Repository 메서드는 null을 반환하지 않고 비어있는 컬렉션을 반환해주므로 Optional로 감싸서 반환할 필요가 없다.

```
// 안 좋음
List<Member> members = team.getMembers();
return Optional.ofNullable(members);

// 좋음
List<Member> members = team.getMembers();
return members != null ? members : Collections.emptyList();
```

### Optional을 필드로 사용 금지

Optional은 필드로 설계되지 않았으며 Serializable하지 않음.

```
// 안 좋음
public class Member {

    private Long id;
    private String name;
    private Optional<String> email = Optional.empty();
}

// 좋음
public class Member {

    private Long id;
    private String name;
    private String email;
}
```

### Optional을 생성자나 메서드 인자로 사용 금지

호출 시마다 Optional 객체를 생성해야 하므로 성능 저하 우려.
인자는 null 허용 후 메서드 내에서 체크. 

```
// 안 좋음
public class HRManager {
    
    public void increaseSalary(Optional<Member> member) {
        member.ifPresent(member -> member.increaseSalary(10));
    }
}
hrManager.increaseSalary(Optional.ofNullable(member));

// 좋음
public class HRManager {
    
    public void increaseSalary(Member member) {
        if (member != null) {
            member.increaseSalary(10);
        }
    }
}
hrManager.increaseSalary(member);
```

### Optional을 컬렉션 원소로 사용 금지

컬렉션에는 많은 원소가 들어갈 수 있음. 따라서 무거운 optional을 사용하지 말고 원소를 꺼낼 때 null 체크 권장

> Map은 getOrDefault(), putIfAbsent(), computeIfAbsent(), computeIfPresent() 처럼 null 체크가 포함된 메서드를 제공합니다.
> Map의 원소로 Optional을 사용하지 말고 Map이 제공하는 메서드를 활용하는 것이 좋습니다.

```
// 안 좋음
Map<String, Optional<String>> sports = new HashMap<>();
sports.put("100", Optional.of("BasketBall"));
sports.put("101", Optional.ofNullable(someOtherSports));
String basketBall = sports.get("100").orElse("BasketBall");
String unknown = sports.get("101").orElse("");

// 좋음
Map<String, String> sports = new HashMap<>();
sports.put("100", "BasketBall");
sports.put("101", null);
String basketBall = sports.getOrDefault("100", "BasketBall");
String unknown = sports.computeIfAbsent("101", k -> "");
```

### of()와 ofNullable() 혼동 주의

of()는 null이 아님이 확실할 때만 사용.
null 가능성이 있으면 ofNullable() 사용.

```
// 안 좋음
return Optional.of(member.getEmail());  // member의 email이 null이면 NPE 발생
// 좋음
return Optional.ofNullable(member.getEmail());

// 안 좋음
return Optional.ofNullable("READY");
// 좋음
return Optional.of("READY");
```

### Optional<T> 대신 OptionalInt, OptionalLong, OptionalDouble

int, long, double 값을 담을 경우 Boxing/Unboxing이 발생하지 않는 OptionalInt, OptionalLong, OptionalDouble 사용.

```
// 안 좋음
Optional<Integer> count = Optional.of(38);  // boxing 발생
for (int i = 0 ; i < count.get() ; i++) { ... }  // unboxing 발생

// 좋음
OptionalInt count = OptionalInt.of(38);  // boxing 발생 안 함
for (int i = 0 ; i < count.getAsInt() ; i++) { ... }  // unboxing 발생 안 함
```

> boxing/unboxing ?
> 값 타입과 참조 타입을 서로 변환해주는 것
