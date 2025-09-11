---
title: "[FastAPI/Troubleshooting] Windows 환경에서 FastAPI + Playwright 실행 시 NotImplementedError 해결"
description: "Windows에서 FastAPI가 asyncio 기반 하위 프로세스를 실행할 때 발생하는 NotImplementedError 문제를 분석하고, reload 옵션과 이벤트 루프 정책 설정 및 WSL2 활용을 통한 해결 방법을 정리합니다."
date: 2025-07-24 15:32:00 +0900
categories: ["development", "frameworks"]
tags: ["fastapi", "playwright", "asyncio", "windows", "troubleshooting"]
pin: false
math: false
mermaid: false
---

Windows 환경에서 FastAPI에서 `playwright` 또는 `asyncio.create_subprocess_exec()` 같은 비동기 하위 프로세스를 실행할 경우 다음과 같은 오류가 발생할 수 있습니다.

```bash
NotImplementedError
```

## 원인

* Windows의 asyncio는 Unix와 달리 하위 프로세스 비동기 실행이 완전히 구현되어 있지 않습니다.  
* ProactorEventLoop와 SelectorEventLoop 간 호환성 문제도 존재합니다.  
* Playwright 내부에서 사용하는 subprocess 관련 API가 Windows에서는 제한됩니다.

## 해결 방법

### 1. FastAPI reload 비활성화

개발 환경에서 `--reload` 옵션은 이벤트 루프 충돌을 유발할 수 있습니다.

```python
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=False)
```

### 2. asyncio 이벤트 루프 정책 지정

Windows에서 적절한 이벤트 루프 정책을 명시해야 합니다.

```python
import asyncio
import sys

if sys.platform.startswith("win"):
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

최신 Windows에서는 다음을 사용할 수 있습니다.

```python
asyncio.set_event_loop_policy(asyncio.WindowsProactorEventLoopPolicy())
```

### 3. WSL2를 통한 우회 실행

Windows의 제약을 피하기 위해 WSL2에서 FastAPI + Playwright를 실행할 수도 있습니다.

```bash
wsl --install -d Ubuntu
sudo apt update && sudo apt upgrade -y
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
pip install fastapi[all] playwright
playwright install
uvicorn app.main:app --host 0.0.0.0 --port 56004
```

> **Tip**: WSL2는 별도의 IP를 가지므로 외부 서비스 접근을 위해 포트포워딩이 필요합니다.

### 4. 포트포워딩 설정

#### Ubuntu 내부

```bash
sudo update-alternatives --config iptables  # → iptables-legacy 선택
sudo iptables -I INPUT 1 -p tcp --dport 56004 -j ACCEPT
```

#### Windows PowerShell

```bash
netsh interface portproxy add v4tov4 listenport=56004 listenaddress=0.0.0.0 connectport=56004 connectaddress=<WSL-IP>
```

---

## 정리

Windows 환경에서 FastAPI와 Playwright를 함께 사용할 때 `NotImplementedError`가 발생하는 주요 원인은 asyncio의 제한입니다.  
reload 옵션을 비활성화하거나 이벤트 루프 정책을 지정하면 대부분 해결되며, 근본적으로는 **WSL2를 활용하는 방식이 안정적**입니다.
