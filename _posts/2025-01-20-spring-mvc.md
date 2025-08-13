---
title: "[spring] Spring MVC가 무엇인지 알아보고 실습해보기"
description: "MVC 패턴은 애플리케이션을 개발할 때 사용하는 디자인 패턴입니다. 이 내용에 대해 자세히 다루고 실습해보는 과정을 요약합니다."
date: 2025-01-20 20:28:00 +0900
categories: [ "programming", "spring" ]
tags: [ "aop", "mvc", "spring", "java", "programming" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-01-20-spring-mvc-springmvc.webp
  alt: "[Spring] Spring MVC가 무엇인지 알아보고 실습해보기"
---

> `AOP`에 대한 개념이 부족하다고 생각이 드시면 [[Spring] AOP](/posts/spring-aop)글을 먼저 읽어보시기를 권장드립니다.
{: .prompt-info}

## MVC 패턴이란?

MVC 패턴은 애플리케이션을 개발할 때 사용하는 디자인 패턴입니다.

* 애플리케이션의 개발 영역을 MVC (Model, View, Controller)로 구분하여 각 역할에 맞게 코드를 작성하는 개발 방식
* MVC 패턴을 도입하면서 **UI 영역과 도메인(비즈니스 로직) 영역으로 구분**되어 서로에게 영향을 주지 않으면서 개발과 유지보수가 가능

> 모델: 애플리케이션의 정보(데이터)를 나타냄  
> 뷰: 텍스트, 체크박스 항목 등과 같은 사용자 인터페이스 요소를 나타냄  
> 컨트롤러: 데이터와 비즈니스 로직 사이의 상호동작을 관리

## MVC 패턴을 사용하는 이유

애플리케이션을 **사용자가 보는 화면(뷰)**, **데이터를 처리하는 로직(모델)**, 그리고 이 둘을 연결하고 제어하는 **컨트롤러**로 나누어 구성하면, 각 요소가 **자신의
역할에만 집중**할 수 있습니다.

**MVC 패턴의 목적**은 각 구성 요소를 분리해 시스템의 결합도를 낮추는 데 있습니다. 이를 통해 유지보수가 쉬워지고, 중복 코드가 줄어들며, 애플리케이션의 확장성과 유연성이
크게 향상됩니다.

## Model, View, Controller

### Model (모델)

Spring MVC 기반의 웹 애플리케이션에서는 클라이언트의 요청을 받아 처리한 결과 데이터를 클라이언트에게 응답으로 돌려줍니다. 이때, 처리 결과 데이터를 **Model**이라고
합니다. Model은 클라이언트 요청에 대한 작업 결과를 담아 전달하는 역할을 합니다.

### View (뷰)

**View**는 Model에 담긴 데이터를 기반으로 사용자에게 보이는 화면을 구성합니다. 웹 브라우저와 같은 애플리케이션 화면에 리소스(Resource)를 제공하는 역할을 하며,
Spring MVC는 다양한 View 기술을 지원합니다.

#### 지원하는 View 기술 예시:

- HTML 페이지 출력
- PDF, Excel 등 문서 형식으로 출력
- XML, JSON 등 특정 포맷으로 변환

---

### Controller (컨트롤러)

**Controller**는 클라이언트 요청을 직접 받아들여 Model과 View 간의 상호작용을 조율하는 역할을 합니다.

주요 역할:

1. 클라이언트 요청을 전달받아 비즈니스 로직을 처리
2. 생성된 Model 데이터를 View로 전달
3. Model과 View를 연결하여 최종적으로 클라이언트에게 응답

컨트롤러는 **Model**과 **View**의 중간다리 역할을 하며, 애플리케이션의 흐름을 관리하는 핵심 요소입니다.

## MVC 패턴과 MVC1, MVC2 아키텍처의 관계

MVC 패턴은 사실 MVC1과 MVC2 아키텍처에서 발전된 형태입니다.

### MVC1

![mvc1](/assets/img/posts/2025-01-20-spring-mvc-mvc1.webp)

MVC1 패턴은 브라우저(사용자)로부터 요청이 들어오면 db로부터 필요한 데이터를 받은 model 객체(java bean)를 jsp 페이지(view)에 담아 응답으로 내보내는
패턴입니다.

MVC1에서는 jsp가 view와 controller역할을 모두 담당하기 때문에 jsp페이지 내 코드가 너무 많이 들어가게 됩니다.
따라서, 코드가 복잡해 가독성이 떨어집니다.

이런 점을 보완하여 controller 역할을 하는 servlet이 추가된 MVC2 패턴이 등장합니다.

### MVC2

![mvc2](/assets/img/posts/2025-01-20-spring-mvc-mvc2.webp)

MVC2패턴은 요청을 하나의 컨트롤러(servlet)가 먼저 받고 서블릿은 요청에 대한 비즈니스 로직을 처리한 후,
이를 jsp파일에 반영하는 역할을 수행합니다.

MVC2는 MVC1패턴보다 구조가 복잡하지만 이런 문제점들을 해결하기 위해 다양한 프레임워크가 발전되어왔고,
그 중 대표적인 것이 바로 **Spring**프레임워크 입니다.

## Spring MVC

Spring 프레임워크에서 MVC2 모델을 더 발전시켜 Spring MVC가 나왔으며 이는 MVC2 모델이 기반인 웹 모듈입니다.

프론트 컨트롤러(Front Controller)가 우선적으로 클라이언트 요청을 받고, 실제 요청 처리는 **개별 컨트롤러 클래스**로 위임합니다.

개별 컨트롤러 클래스는 핸들러(handler)라고도 하며, DI를 통해 생성해둔 bean을 통해 비즈니스 로직 처리 결과를 model에 담아 다시 프론트 컨트롤러로 보냅니다.

프론트 컨트롤러는 **받은 model**을 알맞은 **view 템플릿으로 전달**하여 반영시키고, 최종적으로 클라이언트로 보낼 화면을 결과로 전송합니다.

### 구조

| 구성 요소                                    | 설명                                            |
|------------------------------------------|-----------------------------------------------|
| **DispatcherServlet (Front Controller)** | 모든 요청을 받아 적절한 Controller와 View로 전달            |
| **HandlerMapping**                       | 요청 URL에 맞는 Controller를 결정                     |
| **Controller**                           | 요청 처리 후 결과를 DispatcherServlet에 전달하여 후속작업 진행   |
| **ModelAndView**                         | Controller가 처리 한 데이터 및 화면 정보를 담은 객체           |
| **ViewResolver**                         | Controller가 라턴한 뷰 이름을 바탕으로 처리 결과를 보여줄 View 결정 |
| **View**                                 | Controller의 처리 결과를 보여줄 응답 화면 생성               |

### 실행순서

![spring-mvc-architecture](/assets/img/posts/2025-01-20-spring-mvc-springmvc.webp)

* DispatcherServlet이 요청 수선
* DispatcherServlet은 HandlerMapping에 어느 Controller를 사용할 것인지 문의
* Controller는 ModelAndView객체에 수행결과를 포함하여 DispatcherServlet에 리턴
* View는 결과 정보를 기반으로 화면 표현

## 실습

아래에서 설명할 예제는 단순히 사용자를 등록하고 등록된 사용자를 조회하는 spring 프로젝트입니다.
해당 내용을 바탕으로 고전적인 패턴의 프로젝트를 MVC 패턴으로 수정해보겠습니다.

### 사용자 추가

사용자를 추가하는 코드 수정:

#### 컨트롤러 추가

```java

@Controller
public class UserController {

  private final UserService userService;

  @Autowired
  public UserController(UserService userService) {
    this.userService = userService;
  }

  @GetMapping("/users/new")
  public String createForm() {
    return "users/createUserForm"; // resources/templates/users/createUserForm.html 을 반환
  }
}
```

#### 뷰 생성

```html
<!-- createUserForm.html -->
<form>
  <div class="form-group">
    <!-- 중략 -->
    <label for="email">email</label>
    <input type="text" placeholder="이메일을 입력하세요">
    <!-- 중략 -->
  </div>
  <button type="submit">등록</button>
</form>
```

`http://localhost:8080/usesr/new` 로 접속하면 아래와 비슷한 화면을 확인할 수 있습니다.

![result](/assets/img/posts/2025-01-20-spring-mvc-2025-01-21-13-05-38.webp)

그러나 아직 데이터를 입력하고 등록을 하더라도 어디로 보낼지, 어떻게 보낼지 작성하지 않았기 때문에 아무일도 일어나지 않습니다.

##### 데이터 전송

데이터를 전송하기 위해 `action`에는 어디로 보낼지, `method`는 어떻게 보낼지 (post | get) 작성합니다.

* form 태그에 action, method 속성 추가
* input 태그에 name 속성 추가

```html
<!-- createUserForm.html -->
<form action="/users/new" method="post">
  <div class="form-group">
    <!-- 중략 -->
    <label for="email">email</label>
    <input type="text" id="email" name="email" placeholder="이메일을 입력하세요">
    <!-- 중략 -->
  </div>
  <button type="submit">등록</button>
</form>
```

##### DTO 작성

form 데이터를 받기위해 DTO 객체를 생성합니다.

```java
// UserForm.java
public class UserForm {

  private String email;
  private String name;
  private String password;
  // ... 중략 ...
}
```

##### 폼 데이터 받기

DTO 객체를 UserController 에서 받을 수 있도록 파라미터에 UserForm을 추가하고, 아래와 같은 메서드를 작성합니다.

```java

@PostMapping("/users/new")
public String create(UserForm form) { // UserForm DTO 에 결과값을 담기 위함
  User user = new User();
  user.setEmail(form.getEmail()); // input tag에 name 필드에 작성한 email 값을 파싱하여 가져옵니다.
  // ...중략...
  userService.joinUser(user);
  return "redirect:/";
}
```

전체 코드는 [GitHub](https://github.com/eun2ce/likelion/tree/main/iocexam)에서 확인하실 수 있습니다.
자세한 패치
내역은 [commit](https://github.com/eun2ce/likelion/commit/1e0a414b47e097e5c071a82319d5d44cac5aa6a0#diff-d26b785470ebb2d2919e85b6a7468af9ebd3edfa08724fdc06057b49c0984d36)
을 참고해주세요.
