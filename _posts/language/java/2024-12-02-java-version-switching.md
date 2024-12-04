---
title: "Java 여러가지 버전을 함께 사용하는 방법"
pin: false
math: false
mermaid: false
categories: [ "java", "language" ]
tags: [ "java", "프로그래밍" ]
date: 2024-12-02 10:51:00 +0900
image:
  path: /assets/img/posts/language/java/2024-12-02-java-version-switching/2024120201.png
  alt: "Java 여러가지 버전을 함께 사용하는 방법"
---

프로젝트별로 자바 버전 변경이 필요 한 경우가 있습니다.  
일일히 변경하기 귀찮으니 쉽게 변경할 수 있는 방법에 대해 소개합니다.

```bash
# ~/.zshrc 에 아래 내용을 추가

# java
jdk() {
version=$1
unset JAVA_HOME;
export JAVA_HOME=$(/usr/libexec/java_home -v $version);
export PATH=${PATH}:$JAVA_HOME/bin:
java -version
}

# Java 21
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
export PATH=${PATH}:$JAVA_HOME/bin:
```

default 로 21버전을 사용하고 아래와 같이 터미널에 입력하면 한시적으로 자바 버전을 변경 할 수 있습니다.

![java_version_switching](/assets/img/posts/language/java/2024-12-02-java-version-switching/2024120201.png)
