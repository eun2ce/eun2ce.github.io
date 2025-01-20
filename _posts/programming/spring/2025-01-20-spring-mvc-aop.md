---
title: "[Spring] Spring MVC"
description: ""
date: 2025-01-20 20:28:00 +0900
categories: [ "programming", "spring" ]
tags: [ "aop", "mvc", "spring", "java", "programming", "적용" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/programming/spring/2025-01-20-spring-mvc-aop
  alt: "[Spring] Spring MVC"
---

> `AOP`에 대한 개념이 부족하다고 생각이 드시면 [[Spring] AOP](/posts/spring-aop)글을 먼저 읽어보시기를 권장드립니다.
> {: .prompt-info}

## MVC 패턴이란?

애플리케이션의 확장을 위해 model, view, controller 세 가지 영역으로 분리한 것으로,
컴포넌트의 변경이 다른영역 컴포넌트에 영향을 미치지 않아 유지보수에 용이합니다.
또한, 컴포넌트간의 결합성이 낮아 프로그램 수정이 용이합니다.

## Spring MVC

스프링은 `DI`나 `AOP`와 같은 기능뿐만 아니라 `servlet(서블릿)`기반의 웹개발을 위한 `MVC Framework`를 제공합니다.

### 구성요소

| 구성 요소                                    | 설명                                            |
|------------------------------------------|-----------------------------------------------|
| **DispatcherServlet (Front Controller)** | 모든 요청을 받아 적절한 Controller와 View로 전달            |
| **HandlerMapping**                       | 요청 URL에 맞는 Controller를 결정                     |
| **Controller**                           | 요청 처리 후 결과를 DispatcherServlet에 전달하여 후속작업 진행   |
| **ModelAndView**                         | Controller가 처리 한 데이터 및 화면 정보를 담은 객체           |
| **ViewResolver**                         | Controller가 라턴한 뷰 이름을 바탕으로 처리 결과를 보여줄 View 결정 |
| **View**                                 | Controller의 처리 결과를 보여줄 응답 화면 생성               |

### 실행순서

* DispatcherServlet이 요청 수선
* DispatcherServlet은 HandlerMapping에 어느 Controller를 사용할 것인지 문의
* Controller는 ModelAndView객체에 수행결과를 포함하여 DispatcherServlet에 리턴
* View는 결과 정보를 기반으로 화면 표현

![_spring-mvc-architecture](/assets/img/posts/programming/spring/2025-01-20-spring-mvc-aop/_spring-mvc-architecture.webp)

> 요청 흐름은 위에서 아래방향으로 진행됩니다.
