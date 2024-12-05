---
title: "[intellij] community 에서 spring boot 프로젝트 설정하는 방법"
pin: false
math: false
mermaid: false
categories: [ "programming", "spring" ]
tags: [ "community", "spring", "intellij", "java", "programming" ]
date: 2024-12-04 10:51:00 +0900
image:
  path: /assets/img/posts/programming/spring/2024-12-04-spring-how-to-start-spring-boot-project-in-community-edition-of-intellij-idea/2024120403.png
  alt: "[intellij] community 에서 spring boot 프로젝트 설정하는 방법"
---

intellij 에는 무료버전 community 와 유료버전 ultimate 가 있다.  
community 버전에는 spring initializr 가 따로 없어 그 방법을 메모한다.

## spring initializr 접속

[https://start.spring.io/](https://start.spring.io/)

## 프로젝트 설정

![spring initializr 설정](/assets/img/posts/programming/spring/2024-12-04-spring-how-to-start-spring-boot-project-in-community-edition-of-intellij-idea/2024120401.png)

* group: 프로젝트 정의 및 구분하는데에 필요한 식별자
  * 예: org.apache.maven, org.apache.commons
* artifact: 프로젝트 명
* name: 일반적으로 artifact 를 따름
* 필요한 라이브러리 추가

## workspace 설정

프로젝트를 구동 할 디렉토리에 zip 파일을 옮기고 압축 해제
> finder 나 폴더에 접근해서 압축을 풀어주면 된다.

```bash
// sbb.zip 파일 압축을 ~/works/sbb 에 풀어준다.
$ unzip ~/Downloads/sbb.zip -d ~/works/sbb
```

![spring initializr 설정](/assets/img/posts/programming/spring/2024-12-04-spring-how-to-start-spring-boot-project-in-community-edition-of-intellij-idea/2024120402.png)

## 실행

압축 해제 한 `sbb`디렉토리를 intellij 에서 new project 로 추가

![spring initializr 설정](/assets/img/posts/programming/spring/2024-12-04-spring-how-to-start-spring-boot-project-in-community-edition-of-intellij-idea/2024120403.png)
