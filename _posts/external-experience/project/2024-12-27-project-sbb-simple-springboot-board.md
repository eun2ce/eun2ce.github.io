---
title: "[toy project] spring boot 와 java 를 이용한 게시판 만들기"
date: 2024-12-27 13:01:00 +0900
categories: [ "external-experience", "project" ]
tags: [ "java", "project", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-13-30-27.png
0  alt: "spring boot 와 java 를 이용한 게시판 만들기"
---

> 이 프로젝트는 점프투스트링부트(Jump to Spring Boot) 책을 통해 진행한 SBB(SpringBoot Board) 프로젝트입니다.
> 간단한 게시판 시스템을 구현하며, Spring Boot, Spring Security, JPA 등의 기술을 활용한 웹 애플리케이션을 구축하는 과정에서 많은 실전 경험을 쌓을 수
> 있었습니다.
> 특히, `Spring Security`를 통한 인증 및 권한 관리와 `Gmail`을 이용한 이메일 인증 시스템 구현을 중점적으로 다루어 이야기 합니다.

## Summery

### 기본 기능

- [x] 질문, 답변 등록
- [x] 질문, 답변 수정 및 삭제
- [x] 답변 개수 및 글 추천
  - spring security (with. CSRF)
- [x] 회원가입
- [x] 검색
- [x] 카테고리

### 추가 기능 개발

- [x] 답변 페이징 및 정렬
- [x] 댓글
- [x] 비밀번호 찾기 및 변경
  - `gmail`을 이용한 이메일 인증

### 기타

- `JUnit` 을 이용한 테스트 및 `dummy` 데이터 생성
- `thymeleaf` 사용

## `Spring Security`를 이용한 인증 및 권한 관리

게시판 프로젝트에 처음 spring security 를 적용했더니, 게시글 전체보는 페이지에서도 인증을 요구하는 화면이 나타났습니다.

![login](/assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-13-30-27.png)

## 문제 및 해결

#### `Spring Security` 설정 및 `H2` 콘솔 오류 해결

**Spring Security 설정**

(로그인 없이 게시물을 조회할 수 있도록)`SecurityConfig.java`에서 모든 페이지에 접근을 허용하는 설정을 추가

```
http.authorizeHttpRequests().requestMatchers("/**").permitAll();
```

#### H2 콘솔 403 오류 해결

CSRF 보호로 인해 H2 콘솔 접근 시 403 오류가 발생 H2 콘솔 경로에 대해서만 CSRF 검증을 비활성화

> CSRF 는 사용자가 의도하지 않는 요청을 막기 위한 기능입니다.
> [spring docs](https://docs.spring.io/spring-security/reference/features/exploits/csrf.html#csrf) 에
> 좋은 내용이 있어 첨부합니다.
{: .prompt-info}

```
http.csrf().ignoringRequestMatchers("/h2-console/**");
```

#### H2 콘솔 화면 깨짐 해결

`X-Frame-Options`가 `DENY`로 설정되어 있어 H2 콘솔 화면이 깨지는데, 이를 `SAMEORIGIN`으로 변경하여 해결

```
http.headers().addHeaderWriter(new XFrameOptionsHeaderWriter(XFrameOptionsHeaderWriter.XFrameOptionsMode.SAMEORIGIN));
```

## `gmail`에서 사용자 인증 및 보안 관련 문제

> javax.mail.AuthenticationFailedException
> { : .prompt-danger }

검색해보면 `보안 수준이 낮은 앱에 대한 액세스`설정을 이용한 방법이 많은데, 찝찝하기도하고 이런 해결방법은 내가 생각하는 방향이 아니었다.

해당 로그에 대해 살펴보면 아래와 같다.

[https://support.google.com/mail/troubleshooter/2402620?visit_id=638708727820461404-4184421121&p=BadCredentials&rd=2](https://support.google.com/mail/troubleshooter/2402620?visit_id=638708727820461404-4184421121&p=BadCredentials&rd=2)

즉, 2단계 인증이 필요한 smtp 의 경우 앱 비밀번호를 사용하면 되고  
더 찾아 본 결과 2단계 인증 후 `application.properties` 에 등록해주면 된다.

```
spring.mail.host=smtp.gmail.com
spring.mail.port=587
spring.mail.username=joeun2ce@gmail.com
spring.mail.password={ 앱 비밀번호 }
spring.mail.properties.mail.smtp.auth=true
spring.mail.properties.mail.smtp.starttls.enable=true
```

## 결과물

전체 게시판 글을 확인하고 글을 확인할 수 있습니다.

![check post](/assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-14-24-37.png)

회원가입을 해야하고 해당 이메일은 인증이 가능해야 합니다.

* 가입 된 사용자만 글을 수정하고 댓글을 작성할 수 있습니다.

![modify post](/assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-14-26-06.png)

비밀번호를 변경할 수 있습니다.

![change pw](/assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-14-24-48.png)

* 패스워드 변경 시 이메일 인증이 필요하며, 확인 된 사용자만 패스워드 변경이 이루어 집니다.
  ![change pa (with.email)](/assets/img/posts/external-experience/project/2024-12-27-project-sbb-simple-springboot-board/2024-12-27-14-25-08.png)

### github

* [https://github.com/eun2ce/sbb](https://github.com/eun2ce/sbb)

### 간단 시연 영상

* [https://youtu.be/EFTlsrDutwo](https://youtu.be/EFTlsrDutwo)
