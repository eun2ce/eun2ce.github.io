---
title: "[spring boot] H2 Database \"testdb\" not found, either pre-create it or allow remote database creation 해결"
description: ""
date: 2024-12-05 17:28:00 +0900
categories: [ "programming", "spring" ]
tags: [ "H2", "spring", "java", "programming" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/programming/spring/2024-12-05-spring-h2-database-not-found/2024-12-05-17-50-54.png
  alt: "[Spring Boot] H2 Database \"testdb\" not found, either pre-create it or allow remote database creation 해결"
---

> Database "testdb" not found, either pre-create it or allow remote database creation (not recommended in secure environments) [90149-232] 90149/90149 (Help)
{: .prompt-danger}

* H2 version: `2.3.232`
* Spring Boot: `3.4.0`

## 해결방법

### H2 버전 다운그레이드

H2 버전을 1.4.197 이하로 낮추면 되지만(일반적으로 `1.4.193` 을 많이 사용) 보안상 이슈가 있을 수 있다.

### 데이터베이스 수동 생성

수동 데이터베이스 파일 생성은 
> 수동으로 데이터베이스 파일을 생성하는 방법에 대해 자세한 내용이 궁금하시면, [이 글](https://h2database.com/html/tutorial.html#creating_new_databases)을 참고해주세요.
{: .prompt-info }

### 더미 Entitiy 생성으로 Db 생성 유도

`JPA`를 사용해 `ddl-auto` 기능을 활용해 해결

#### bundle.gradle

```java
dependencies {
  // bundle.gradle 에 추가
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

#### application.properties

`create-drop`으로 `ddl-auto` 기능 사용

```
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.H2Dialect
jpa.hibernate.ddl-auto=create-drop
```

#### 테이블 추가

`JPA` 가 자동으로 테이블을 만들 수 있을 정도로 추가
> 어차피 인메모리에서 사용되기 때문에 재 실행시 초기화

```java
@Entity
public class Question {
  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Integer id;
}
```

## 결과

### test connection

![img](/assets/img/posts/programming/spring/2024-12-05-spring-h2-database-not-found/2024-12-05-17-52-27.png)
성공

### connection

![img](/assets/img/posts/programming/spring/2024-12-05-spring-h2-database-not-found/2024-12-05-17-51-29.png)

정상적으로 실행되고 경로에 db 파일이 생성 된 것을 확인할 수 있다.

![img](/assets/img/posts/programming/spring/2024-12-05-spring-h2-database-not-found/2024-12-05-17-50-54.png)
