---
title: "[ubuntu] pipx로 uv 패키지 매니저 설치 및 설정 (Ubuntu 22.04)"
date: 2025-07-09 13:20:00 +0900
categories: [ "etc", "else" ]
tags: [ "uv", "pipx", "python", "virtualenv", "ubuntu", "venv" ]
pin: false
math: false
mermaid: false
---

Ubuntu 2x.04 LTS에서 [uv](https://github.com/astral-sh/uv)를 `pipx`를 사용하여 설치하는 방법을 간단히 정리합니다.

## 사전 조건

- Python 3.10 이상이 설치되어 있어야 합니다.
- 루트(root) 계정 또는 `sudo` 권한

---

## 설치 순서

### 1. 패키지 업데이트 및 pipx 설치

```bash
sudo apt update
sudo apt install python3-pip -y
sudo apt install pipx -y
```

> `pipx`는 python 패키지를 격리된 환경에서 설치해주는 실행기입니다.

---

### 2. uv 설치

```bash
pipx install uv
```

설치 후 아래와 같은 메시지가 나올 수 있습니다:

```
Note: '/root/.local/bin' is not on your PATH environment variable.
```

이 경우 다음 단계를 진행합니다.

---

### 3. PATH 설정

```bash
pipx ensurepath
source ~/.bashrc
```

또는 새로운 터미널을 열어야 적용됩니다.

---

### 4. 설치 확인

```bash
uv --version
```

설치가 완료되었으면 `uv` 버전이 출력됩니다.

---

## 참고

- `pipx`는 가상 환경을 자동으로 생성해주기 때문에 `uv` 외 다른 CLI 도구를 설치할 때도 유용합니다.
- root 사용자일 경우 `~/.bashrc` 대신 `/root/.bashrc`를 사용합니다.

```bash
source /root/.bashrc
```
