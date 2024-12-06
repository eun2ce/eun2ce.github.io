---
title: "Java 형변환"
date: 2024-12-03 16:00:00 +0900
categories: [ "language", "java" ]
tags: [ "java", "프로그래밍" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/language/java/2024-12-03-java-type-conversion/2024120301.png
  alt: "Java 형변환"
---

#### int to String

```java
String str = Integer.toString(i);
String str = "" + i;
```

#### String to int

```java
int i = Integer.parseInt(str);
int i = Integer.valueOf(str).intValue();
```

#### double to String

```java
String str = Double.toString(d);
```

#### long to String

```java
String str = Long.toString(l);
```

#### float to String

```java
String str = Float.toString(f);
```

#### String to double

```java
double d = Double.valueOf(str).doubleValue();
```

#### String to long

```java
long l = Long.valueOf(str).longValue();
long l = Long.parseLong(str);
```

#### String to float

```java
float f = Float.valueOf(str).floatValue();
```

#### decimal to binary

```java
String binstr = Integer.toBinaryString(i);
```

#### decimal to hexadecimal

```
String hexstr = Integer.toString(i, 16);
String hexstr = Integer.toHexString(i);
Integer.toHexString(0x10000 | i).substring(1).toUpperCase();
```

#### hexadecimal(String) to int

```java
int i = Integer.valueOf("B8DA3", 16).intValue();
int i = Integer.parseInt("B8DA3", 16);
```

#### ASCII Code to String

```
String char = new Character((char)i).toString();
```

#### Integer to ASCII Code

```java
int i = (int) c;
```

#### Integer to boolean

```java
boolean b = (i != 0);
```

#### boolean to Integer

```java
int i = (b) ? 1 : 0;
```
