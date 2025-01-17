---
title: "[Spring] IoC와 DI: 스프링에서의 객체 관리의 핵심 개념"
description: ""
date: 2025-01-17 13:52:00 +0900
categories: [ "programming", "spring" ]
tags: [ "di", "ioc", "spring", "java", "programming" ]
pin: false
math: false
mermaid: false
---

## Bean의 정의

Bean은 Spring IoC Container에 의해 관리되는 자바 객체 입니다.

이 객체들은 Spring이 생성, 초기화, 소멸 등을 **관리**하며, 개발자는 객체의 생명 주기를 직접 관리할 필요가 사라집니다.
즉, 스프링은 개발자가 생성한 설정 파일(xml <bean/>)이나 어노테이션(@Component)을 읽어 빈을 생성합니다.

* Bean의 이름은 컨테이너 안에서 유일성을 가져야 합니다. (한 컨테이너 안에 aService라는 빈이 2개 이상일 수 없음) 컨테이너가 빈을 찾지 못할 경우에는 @Qualifier를 통해 스프링 컨테이너가 주입할 빈을 알려줄 수 있습니다.
  * [NoUniqueBeanDefinitionException 발생 시 요령](https://www.baeldung.com/spring-qualifier-annotation)

개발자가 객체를 관리하지 않고, Spring IoC 컨테이너에게 위임하고,
특정 클래스(Controller, Service, Repository)만 빈으로 등록하는 이유는 **IoC (Inversion of Control, 제어의 역전)** 때문이라고 할 수 있습니다.

## IoC와 DI

### 정의

* DI(Dependency Injection, 의존성 주입): 객체 간의 의존관계를 주입
  * e.g) User 객체의 생성자 안에 name 인자
* 제어: 객체(object)간의 의존관계를 설정하고 생명주기(생성, 소멸)를 관리하는 행위
* IoC(Inversion of Control, 제어의 역전): 객체 (class 안에서)가 능동적으로 제어하지 않고 수동적으로 다른 것으로부터 제어를 당할 수 있게 **제어 행위 주체를 역전**시킴

##### IoC는 DI가 아니며, IoC는 객체 간의 의존 관계와 제어를 외부로 위임하는 개념이고, DI는 그 구현 방식 중 하나일 뿐입니다.

스프링은 객체 간의 의존성 주입을 각 객체들이 스스로 클래스 안에서 코드로 주입하도록 두지 않고, IoC 컨테이너가 주도적으로 할 수 있게 위임할 수 있습니다.

객체를 빈(bean)으로 등록하고, 의존성을 명시해 주면 스프링 IoC 컨테이너가 개발자 대신 의존성 주입을 해줍니다. (이것을 제어의 역전이라 함)


즉, 제어의 역전이라 함은 아래와 같이 정리할 수 있습니다.

##### <span style="background-color:#fff5b1">객체간의 연관관계(의존성,생명주기)를 `java code`로 `class` 파일 안에서 객체가 능동적으로 할 수 있게 하지 않고, 의존관계를 사전에 명시한 다음 그 객체를 `BeanFactory`에 `bean`으로 등록하여 스프링 IoC 컨테이너가 할 수 있게 위임한다. </span>

간단한 예시를 살펴봅니다.

```java
public class Major {

  private String name;

  public Major() {
  }

  public Major(String name) {
    this.name = name;
  }
}

@Component
public class Student {

  private String name;

  private Major major;

  public Student() {
  }

  public Student(String name, Major major) {
    this.name = name;
    this.major = major;
  }
}

@Configuration
@ComponentScan(basePackageClasses = Student.class)
public class AppConfig {
  @Bean
  public Major getMajor() {
    return new Major("CS");
  }
}
```

빈을 가져와보면 CS Major의 의존성이 주입되어 있습니다.

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Student student = context.getBean("student", Student.class);
student.getMajor().getName();  // CS
```

실제로 VO 클래스는 스프링 빈으로 잘 등록하지 않기때문에 의의가 있는 예제를 하나 더 작성합니다.

```java
public class UserController {
  @RequestMapping("/")
  @ResponseBody
  public void test()  {
    UserRepository userRepository =  new UserRepository();
    UserService userService = new UserService(userRepository);

    userService.viewMember();
  }
}

public class UserService {

  private UserRepository userRepository;

  public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
  }

}

public class UserRepository {
}
```

#### 문제점

* UserRepository 내용은 런타임에 바뀔 일이 없는데 매번 새로운 객체로 생성 (성능저하)
* UserService 객체를 만들 때마다 매번 UserRepository 의존성을 주입해줘야하는 번거로움 (반복적인 코드)

#### 개선

스프링 빈으로 등록하면

```java
public class UserController {

    @Autowired // 의존성 주입
    private UserService userService;

    @RequestMapping("/")
    @ResponseBody
    public void test()  {
       userService.viewMember();
    }
}

@Service // 빈 등록
public class UserService {

    @Autowired // 의존성 주입
    private UserRepository userRepository;
}

@Repository // 빈 등록
public class UserRepository {
}
```

> @Service, @Repository 어노테이션으로 빈을 등록하고, @Autowired로 의존성 주입

* 싱글톤 패턴으로 해당 객체를 관리
* 빈으로 등록된 객체에 한해 쉽게 의존성 주입 가능

### 의존성 주입(DI)의 예시

의존성 주입 방법에는 크게 3가지가 있고 스프링의 경우 생성자를 통해 주입하는 방법을 추천합니다.
(위에서 든 개발자가 직접 의존성을 주입하는 방법 제외)

#### 필드

```java
@Service
public class UserService {

    @Autowired 
    private UserRepository userRepository;
    
}
```

#### Setter-Based DI (setter method 기반 의존성 주입)

* 주로 optional 하게 의존성을 주입하는 경우 사용
* 의존 대상 객체가 null 이어도 서비스 구동에 문제가 없기 때문에, 런타임 NullPointerException 주의
  * 방지를 위해서는 default 값 설정할 것

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

}
```

#### Constructor-Based DI (생성자 기반 의존성 주입) (<span style="color:#C0FFFF"> recommended</span>)

```java
@Service
public class UserService {

    private UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

스프링은 Bean의 의존성 주입에 한해 아래와 같은 이점을 취할 수 있어 이 방식을 추천합니다.

* 의존 객체를 불변 객체로 생성 가능 (final)
* 순환 참조 사전 감지
* 의존 객체 not null (NullPointerException 방지)

```java
@Service
public class UserService {

    private final UserRepository userRepository; // final

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository; // service가 repository 참조
    }
}

@Repository
public class UserRepository {

    private final UserService userService;
    
    @Autowired
    public UserRepository(UserService userService) {
        this.userService = userService; // 서비스 시작 시 The dependencies of some of the beeans in the application context form a cycle 에러 발생
    }
}
```

## 결론

Spring에서 **Bean**은 IoC 컨테이너가 관리하는 객체입니다. **IoC**는 객체의 생명주기와 의존성 관계를 외부 컨테이너에 맡기는 개념이고, **DI**는 그 구현 방식으로, 의존성을 자동으로 주입해주는 방법입니다.

### 마치며

객체가 서론 의존하는 방식은 객체 간의 결합도를 크게 낮출 수 있어 유용하다는 사실을 알게 되었습니다.
`Student`클래스가 `Major`를 직접 생성하거나 관리하는 대신, 외부에서 `Major`를 주입받는 방식으로
개발자가 `Student`객체를 변경하지 않아도 `Major`의 구현을 자유롭게 변경할 수 있습니다.

이처럼 스프링 IoC 컨테이너는 **객체 간의 관계를 관리**하고, **유연하게 수정**할 수 있는 장점이 있습니다.
실제로 스프링을 사용하면서 DI와 Bean 관리는 객체 지향 설계에서 중요한 부분임을 깨닫고, 프로젝트에서 유용하게 사용할 수 있을 것 같다는 확신이 들었습니다.
