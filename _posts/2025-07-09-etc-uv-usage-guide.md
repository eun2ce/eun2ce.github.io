---
title: "[python] uv 사용법 완벽 정리"
date: 2025-07-09 13:21:00 +0900
categories: [ "etc", "else" ]
tags: [ "uv", "pip", "venv", "dependency", "python-package" ]
pin: false
math: false
mermaid: false
---

Python 프로젝트에서 [uv](https://github.com/astral-sh/uv) 를 사용해 개발/테스트 의존성을 효율적으로 관리하는 방법을 정리합니다.  
[PEP 621](https://peps.python.org/pep-0621/) 기반의 `pyproject.toml` 구조를 기준으로 설명합니다.

# pyproject.toml 예시

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "Example Python application using uv"
readme = "README.md"
requires-python = ">=3.10"
dependencies = [
    # 예: "requests>=2.31.0", "pydantic>=2.6.0"
]

[dependency-groups]
dev = ["ipykernel>=6.29.5", "ruff>=0.11.13"]
test = ["pytest>=8.0.0", "pytest-asyncio>=0.23.0", "httpx>=0.27.0"]
```

# 설치 명령어

## 기본 설치

```bash
uv pip install -r requirements.txt
# 또는 pyproject.toml 기반 설치
uv pip install .
```

## 개발 환경용 패키지 설치 (`[dev]` 그룹)

```bash
uv pip install --dev
```

또는 `uv add` 명령으로 직접 추가:

```bash
uv add --dev ipykernel ruff
```

## 테스트 환경용 패키지 설치 (`[test]` 그룹)

```bash
uv pip install --group test
```

또는 개별 패키지 추가:

```bash
uv add --group test pytest pytest-asyncio httpx
```

# 주요 명령 요약

| 명령어                            | 설명                                     |
|----------------------------------|------------------------------------------|
| `uv add`                         | 의존성 추가                              |
| `uv add --dev foo`              | 개발용(dev) 패키지 추가                  |
| `uv add --group test bar`       | 테스트(test) 그룹 패키지 추가            |
| `uv pip install --dev`          | dev 그룹 패키지 일괄 설치                |
| `uv pip install --group test`   | test 그룹 패키지 일괄 설치               |
| `uv pip sync`                   | `pyproject.lock` 기준 설치 동기화        |
| `uv pip update`                 | 패키지 최신 버전으로 업그레이드         |

# 기타 참고

- `uv`는 Poetry나 Pipenv보다 훨씬 빠른 설치 속도와 캐시 전략을 갖고 있음.
- 그룹별 의존성을 명확히 구분하면 CI/CD 및 로컬 개발 환경 구성에서 편리함.

