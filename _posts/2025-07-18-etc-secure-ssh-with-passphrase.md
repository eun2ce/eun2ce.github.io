---
title: "[SSH/GitHub] SSH 키 생성: 패스프레이즈 없는 방법 vs 매번 인증하는 방법"
date: 2025-07-18 12:42:00 +0900
categories: [ "development", "dev&tools" ]
tags: [ "ssh", "ssh-keygen", "ssh-agent", "github", "git", "passphrase" ]
pin: false
math: false
mermaid: false
---

git ssh 설정하는 방법에 대해 작성합니다.

## ssh 키에 패스 프레이즈 없이 생성하는 방법

### 키 생성

```bash
(base) ubuntu@a100-80g-4:~/.ssh$ ssh-keygen -t ed25519 -f ~/.ssh/joeun2ce -C "joeun2ce@gmail.com"
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/ubuntu/.ssh/joeun2ce
Your public key has been saved in /home/ubuntu/.ssh/joeun2ce.pub
The key fingerprint is:
SHA256:VBPmvx/9I+4yH+ofvajmLJDWIBioyWPbHvYfMC+G6QQ joeun2ce@gmail.com
The key's randomart image is:
+--[ED25519 256]--+
|  .       =.     |
| . .     + .     |
|o.  o   . .      |
         ...
|   .  ..  =*O*o.o|
+----[SHA256]-----+
```

### github에 공개키 등록

```bash
$ cat ~/.ssh/joeun2ce.pub
// 출력된 내용을 GitHub > Settings > SSH and GPG keys > New SSH Key 에 붙여넣기
```

### ssh config 설정

* `~/.ssh/config` 파일이 없다면 만들고 추가

```bash
Host github-eun2ce
  HostName github.com
  User git
  IdentityFile ~/.ssh/joeun2ce
  AddKeysToAgent yes
```

### ssh-agent에 등록

```bash
(base) ubuntu@a100-80g-4:~/.ssh$ eval "$(ssh-agent -s)"
Agent pid {pid}
(base) ubuntu@a100-80g-4:~/.ssh$ ssh-add ~/.ssh/joeun2ce
Identity added: /home/ubuntu/.ssh/joeun2ce (eun2ce.xxx@gmail.com)
```

### 사용

```bash
git remote set-url origin git@github-eun2ce:yourusername/repo.git
```


##  ssh 키에 패스 프레이즈 있이 생성하는 방법

* 반대로 매번 사용하고 싶지 않다면 passphrase 를 입력하고 ssh-agent에 등록하지 않으면 됩니다.

```bash
(base) ubuntu@a100-80g-4:~/.ssh$ ssh-keygen -t ed25519 -f ~/.ssh/joeun2ce -C "joeun2ce@gmail.com"
Enter file in which to save the key: ~/.ssh/joeun2ce_ed25519
Enter passphrase: ********
Enter same passphrase again: ********

ssh-add ~/.ssh/joeun2ce_ed25519  # 하지말것
```

### 이미 캐싱 키 등록했다면 취소하는 방법

```bash
// 등록되어있음
(base) ubuntu@a100-80g-4:~$ ssh-add -l
256 SHA256:+g8xxxxxVQabuM3V5CF/hxxxxla@gmail.com (ED25519)

// 모든 키 삭제
$ ssh-add -D

// 특정 키 삭제
$ ssh-add -d ~/.ssh/joeun2ce_ed25519
```