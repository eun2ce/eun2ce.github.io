---
title: "[git] tistory 블로그 글 github 에 올리는 방법"
description: "git Action 을 이용한 블로그와 github 동기화하는 방법을 다룹니다."
date: 2024-12-06 15:32:00 +0900
categories: [ "programming", "git" ]
tags: [ "blog", "crawl", "crawling", "git", "python", "tistory", "깃", "깃헙", "블로그", "크롤링" ]
pin: false
math: false
mermaid: false
---

블로그 포스팅을 `github`에 자동으로 올리는 방법을 알아보려고 합니다. 이 글은 `tistory` 블로그를 기준으로 설명합니다.

## github 레포(repository) 생성

블로그 글을 남길 레포지토리를 생성합니다.

![repository 생성](/assets/img/posts/programming/git/2024-12-04-git-using-gitaction-with-tistory/2024-12-06-15-10-00.png)

## git actions 생성

특정 주기로 블로그 글을 가져올 수 있도록하려면 아래와 같은 작업이 필요합니다.

### 생성한 레포지토리에서 Actions 클릭

![actions](/assets/img/posts/programming/git/2024-12-04-git-using-gitaction-with-tistory/2024-12-06-15-18-00.png)

### workflow 파일 작성

`Simple workflow` 클릭하고 아래 내용을 복사해줍니다. (자세한 설명은 주석을 참고해주세요.)

```yaml
name: tistory updater

on:
  push:
    branches: [ main ] # 어느 브랜치에 push
  pull_request:
    branches: [ main ] # 어느 브랜치에 pull request
  schedule: # https://en.wikipedia.org/wiki/Cron
    - cron: "0 0 */1 * *" # 매일
    # - cron: "*/1 * * * *" # 1분마다 (테스트용도로만 사용하세요)

jobs:
  build:
    runs-on: ubuntu-latest # ubuntu 최신 버전 사용
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.7
      uses: actions/setup-python@v2
      with:
        python-version: '3.7' # 사용하고 싶은 버전 사용해도 상관 x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install feedparser
    - name: Update README
      run: |
        python main.py
    - name: Commit README
      run: |
        git add .
        git diff
        git config --local user.email "eun2ce@gihtub.com" # commit 이력에 남길 email
        git config --local user.name "eun2ce" # commit 이력에 남길 유저 명
        git commit -m "docs: updated my tistory blog"
        git push
```

`actions` > `tistory uploader` 에 들어가면 다음과 같이 블로그 글이 업로드 된 것을 확인할 수 있습니다.

![result](/assets/img/posts/programming/git/2024-12-04-git-using-gitaction-with-tistory/2024-12-06-15-38-59.png)


작성 한 내용이 어떻게 나타나는지 확인하려면 actions 에서 `tistory uploader`를 클릭하면 됩니다.

![check](/assets/img/posts/programming/git/2024-12-04-git-using-gitaction-with-tistory/2024-12-06-15-26-38.png)

## example

[https://github.com/eun2ce/tistory](https://github.com/eun2ce/tistory)
