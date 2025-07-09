---
title: "[python] uv 사용법 완벽 정리"
date: 2025-07-09 13:21:00 +0900
categories: [ "etc", "devops", "python" ]
tags: [ "uv", "pip", "venv", "dependency", "python-package" ]
pin: false
math: false
mermaid: false
---

# 소개

[`uv`](https://github.com/astral-sh/uv)는 Python의 종합 패키지 관리 툴입니다. 기존 `pip`, `pip-tools`, `virtualenv`, `pip-audit` 등을 통합하여 **빠르고 안전한 패키지 관리**를 제공합니다. Rust로 작성되어 매우 빠르며, `PEP 508/517/518`을 완벽하게 지원합니다.

---

# 설치

## pipx를 통해 설치

```bash
pipx install uv
```

또는 수동 설치:

```bash
curl -Ls https://astral.sh/uv/install.sh | bash
```

설치 확인:

```bash
uv --version
```

---

# 주요 기능 요약

| 기능            | 설명                                         |
|-----------------|----------------------------------------------|
| `uv venv`       | 가상환경 생성 (`python -m venv` 대체)       |
| `uv pip`        | pip 대체, 패키지 설치                        |
| `uv pip freeze` | 의존성 목록 저장 (`requirements.txt` 등)    |
| `uv pip sync`   | lock 기반 재현 설치 (pip-tools 기능)         |
| `uv pip audit`  | 보안 취약점 검사 (`pip-audit` 대체)         |
| `uv pip index`  | PyPI mirror 설정 등                          |

---

# 사용법

## 1. 가상환경 생성 및 활성화

```bash
uv venv
source .venv/bin/activate
```

## 2. 패키지 설치

```bash
uv pip install requests pandas
```

## 3. 패키지 제거

```bash
uv pip uninstall pandas
```

## 4. requirements.txt 저장

```bash
uv pip freeze > requirements.txt
```

## 5. requirements.txt로 설치

```bash
uv pip install -r requirements.txt
```

## 6. 보안 감사

```bash
uv pip audit
```

---

# 고급 기능

## sync 설치 (lock 기반 재현성 확보)

`uv`는 `pip-tools`처럼 `requirements.lock.txt` 를 기반으로 환경을 재현할 수 있습니다.

```bash
uv pip sync requirements.txt
```

## PyPI 미러 변경

```bash
uv pip index set https://pypi.org/simple
```

## 캐시 비우기

```bash
uv cache clean
```

---

# 자주 사용하는 커맨드 정리

```bash
uv venv                       # 가상환경 생성
uv pip install -r req.txt    # 패키지 설치
uv pip freeze > req.txt      # 의존성 저장
uv pip sync req.txt          # lock 기반 설치
uv pip audit                 # 보안 점검
```

---

# 참고 링크

- https://github.com/astral-sh/uv
- https://astral.sh/blog/posts/introducing-uv/
