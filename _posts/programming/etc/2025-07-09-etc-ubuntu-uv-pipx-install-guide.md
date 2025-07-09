---
title: "[ubuntu] pipx로 uv 패키지 매니저 설치 및 설정 (Ubuntu 22.04)"
date: 2025-07-09 17:30:00 +0900
categories: [ "etc", "devops", "python" ]
tags: [ "uv", "pipx", "python", "virtualenv", "ubuntu", "venv" ]
pin: false
math: false
mermaid: false
---

# 소개

[`uv`](https://github.com/astral-sh/uv)는 Rust로 작성된 고성능 Python 패키지 매니저입니다. Python 프로젝트의 의존성 설치 속도를 획기적으로 개선하며, PEP 508/517/518 을 따릅니다.

이 가이드에서는 `pipx`를 통해 시스템 전체에 영향 없이 `uv`를 설치하고 사용하는 방법을 설명합니다.

---

# 1. pipx 설치

먼저 `pipx`를 설치합니다:

```bash
sudo apt update
sudo apt install python3-pip python3-venv -y

pip install --user pipx
python3 -m pipx ensurepath
```

```bash
# 쉘 재시작 후, 적용 확인
exec $SHELL
```

설치 확인:

```bash
pipx --version
```

---

# 2. uv 설치

```bash
pipx install uv
```

설치 경로 확인:

```bash
which uv
uv --version
```

> uv는 `pip`, `virtualenv`, `pip-tools`, `pip-audit` 등을 통합한 대체 도구입니다.

---

# 3. uv 기본 사용법

## 가상환경 생성 및 활성화

```bash
uv venv
source .venv/bin/activate
```

## 패키지 설치

```bash
uv pip install requests
```

## requirements.txt 생성

```bash
uv pip freeze > requirements.txt
```

---

# 4. 기존 프로젝트에 적용

기존 프로젝트에 적용 예시:

```bash
cd my_project/
uv venv
source .venv/bin/activate
uv pip install -r requirements.txt
```

---

# 요약

| 항목             | 설명                                      |
|------------------|-------------------------------------------|
| 패키지 관리자    | uv (`pip`, `venv`, `pip-tools` 대체)     |
| 설치 방식        | `pipx`로 글로벌 설치                      |
| 가상환경 생성    | `uv venv`                                 |
| 의존성 설치      | `uv pip install -r requirements.txt`      |
| 요구사항 저장    | `uv pip freeze > requirements.txt`        |

---

# 참고 링크

- https://github.com/astral-sh/uv
- https://pypa.github.io/pipx/
