---
title: "[docker] 컨테이너 이미지 일괄 삭제"
date: 2024-12-17 00:33:00 +0900
categories: [ "ops", "docker" ]
tags: [ "container", "docker", "image" ,"programming" ]
pin: false
math: false
mermaid: false
---

## 컨테이너 일괄 삭제

```bash
$ docker rm `docker ps -a -q`
```

## docker 이미지 일괄 삭제

```bash
$ docker rmi `docker images -q`
```

## 안쓰는 컨테이너, 이미지 삭제

```bash
$ docker image prune -a
```

## 사용하지 않는 컨테이너, 이미지, 볼륨, 네트워크 삭제

```bash
$ docker system prune
```
