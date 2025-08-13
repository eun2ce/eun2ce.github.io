---
title: "[git] git submodule - repo 의 특정 폴더를 다른 repo 하위 모듈로 추가하는 방법"
description: "git Action 을 이용한 블로그와 github 동기화하는 방법을 다룹니다."
date: 2024-12-18 10:47:00 +0900
categories: [ "etc", "git" ]
tags: [ "blog", "crawl", "crawling", "git", "python", "tistory", "깃", "깃헙", "블로그", "크롤링" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2024-12-18-git-add-a-specific-folder-from-a-git-repo-as-a-git-submodule-2024-12-18-10-24-34.wepb
  alt: "repo 의 특정 폴더를 다른 repo 하위 모듈로 추가하는 방법"
---

## git submodule 특정 폴더 추가하는 방법

```bash
$ (~/works) > tree .
.
├── likelion
│   ├── README.md
│   └── lionbank <--- 여기 디렉토리만
└── reivew-team6
    ├── README.md
    └── bankproj
        └── lionbank-eun2ce <--- 여기 밑에 추가
            └── lionbank
```

### git submodule 추가

```bash
$ (~/works/reivew-team6/bankproj) > git submodule add https://github.com/eun2ce/likelion.git lionbank-eun2ce
Cloning into '~/works/reivew-team6/bankproj/lionbank-eun2ce'...
remote: Enumerating objects: 36, done.
remote: Counting objects: 100% (36/36), done.
remote: Compressing objects: 100% (24/24), done.
remote: Total 36 (delta 1), reused 36 (delta 1), pack-reused 0 (from 0)
Receiving objects: 100% (36/36), 63.40 KiB | 15.85 MiB/s, done.
Resolving deltas: 100% (1/1), done.
```

### 메인 프로젝트와 연결

서브모듈 내부의 .git 디렉토리를 부모 저장소로 흡수

```bash
$ (~/works/reivew-team6/bankproj) >  git submodule absorbgitdirs
```

### sparse-checkout 설정

부분 체크아웃이 가능하도록 설정

```bash
$ (~/works/reivew-team6/bankproj) >  git -C lionbank-eun2ce config core.sparseCheckout true
```

![submodule](/assets/img/posts/2024-12-18-git-add-a-specific-folder-from-a-git-repo-as-a-git-submodule-2024-12-18-10-24-34.wepb)

### 특정 디렉토리 지정

```bash
$ (~/works/reivew-team6) > echo "lionbank/*" > .git/modules/bankproj/lionbank-eun2ce/info/sparse-checkout
```

### git update

```bash
$ (~/works/reivew-team6) > git submodule update --force --checkout bankproj/lionbank-eun2ce
```

## git repo

* reivew-team6 [eun2ce/reivew-team6](https://github.com/eun2ce/reivew-team6/tree/main/bankproj)
* likelion [eun2ce/likelion](https://github.com/eun2ce/likelion)
