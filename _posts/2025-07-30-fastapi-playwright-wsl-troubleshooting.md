---
title: "[fastapi/trubleshooting] fastapi + playwirght NotImplementedError"
description: "fastapi에서 asyncio 하위 프로세스를 실행하면 구현되지 않은 오류가 발생"
date: 2025-07-24 15:32:00 +0900
categories: [ "etc", "else" ]
tags: [ "fastapi", "playwright", "asyncio", "troubleshooting" ]
pin: false
math: false
mermaid: false
---


Windows 환경에서 FastAPI에서 `playwright` 또는 `asyncio.create_subprocess_exec()` 등 비동기 하위 프로세스를 실행할 경우 다음과 같은 에러가 발생

```bash
ERROR:    Exception in ASGI application
Traceback (most recent call last):
  File "E:\pycharm\mlsproject\venv\lib\site-packages\uvicorn\protocols\http\httptools_impl.py", line 375, in run_asgi
    result = await app(self.scope, self.receive, self.send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\uvicorn\middleware\proxy_headers.py", line 75, in __call__
    return await self.app(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\fastapi\applications.py", line 208, in __call__
    await super().__call__(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\applications.py", line 112, in __call__
    await self.middleware_stack(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\middleware\errors.py", line 181, in __call__
    raise exc
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\middleware\errors.py", line 159, in __call__
    await self.app(scope, receive, _send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\middleware\cors.py", line 84, in __call__
    await self.app(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\exceptions.py", line 82, in __call__
    raise exc
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\exceptions.py", line 71, in __call__
    await self.app(scope, receive, sender)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\routing.py", line 656, in __call__
    await route.handle(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\routing.py", line 259, in handle
    await self.app(scope, receive, send)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\starlette\routing.py", line 61, in app
    response = await func(request)
  File "E:\pycharm\mlsproject\venv\lib\site-packages\fastapi\routing.py", line 226, in app
    raw_response = await run_endpoint_function(
  File "E:\pycharm\mlsproject\venv\lib\site-packages\fastapi\routing.py", line 159, in run_endpoint_function
    return await dependant.call(**values)
  File "E:\pycharm\mlsproject\src\backend\app\api\api.py", line 202, in subprocess_test
    parsed_json = await(probe_video_file(Path(r"G:\ffmpeg testing\ffmpeg\ffprobe.exe"),
  File "E:\pycharm\mlsproject\src\backend\app\ffmpeg\ffprobe.py", line 26, in probe_video_file
    proc = await asyncio.create_subprocess_exec(
  File "C:\Users\user\AppData\Local\Programs\Python\Python311\lib\asyncio\subprocess.py", line 218, in create_subprocess_exec
    transport, protocol = await loop.subprocess_exec(
  File "C:\Users\user\AppData\Local\Programs\Python\Python311\lib\asyncio\base_events.py", line 1652, in subprocess_exec
    transport = await self._make_subprocess_transport(
  File "C:\Users\user\AppData\Local\Programs\Python\Python311\lib\asyncio\base_events.py", line 493, in _make_subprocess_transport
    raise NotImplementedError
NotImplementedError
INFO:     127.0.0.1:56004 - "GET /test/subprocess HTTP/1.1" 500 Internal Server Error
```

* [https://github.com/fastapi/fastapi/discussions/8391](https://github.com/fastapi/fastapi/discussions/8391)

## 원인

* Windows의 asyncio 구현은 Unix와 달리 하위 프로세스를 비동기적으로 실행하는 기능이 완전하게 구현되어 있지 않음
* 특히 ProactorEventLoop와 SelectorEventLoop 간 호환성 이슈 존재
* Playwright 내부에서도 async_execute, subprocess_exec 같은 API가 Windows에서는 제한됨

## 해결방법

### fastapi reload 비활성화

```python
// reload 비활성화
if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=False)
```

* --reload=True는 내부적으로 watchgod 또는 watchfiles를 사용하여 여러 이벤트 루프를 생성할 수 있으므로 충돌 발생


### asyncio 이벤트 루프 정책 명시

```python
import asyncio
import sys

if sys.platform.startswith("win"):
    asyncio.set_event_loop_policy(asyncio.WindowsSelectorEventLoopPolicy())
```

최신 윈도우는:

```python
asyncio.set_event_loop_policy(asyncio.WindowsProactorEventLoopPolicy())
```

###  wsl2 우회

> 안타깝게도 필자는 위에껄 나중에 알아봐서 아래 방법으로 해결함

* Windows에서 발생하는 NotImplementedError를 회피하기 위해 WSL2에 Ubuntu를 설치 후 FastAPI + Playwright 실행
* 이후 iptables 및 netsh를 이용하여 포트 포워딩 설정

```bash
# WSL2 설치 (Windows PowerShell 관리자 권한)
wsl --install -d Ubuntu

# Ubuntu 초기 설정 후 패키지 업데이트
sudo apt update && sudo apt upgrade -y

# Python 가상환경 생성 및 Playwright 설치
sudo apt install python3-venv -y
python3 -m venv venv
source venv/bin/activate
pip install fastapi[all] playwright
playwright install

uvicorn app.main:app --host 0.0.0.0 --port 56004
```

wsl은 별도의 ip를 갖기 때문에 외부로 서비스 하기위해서는 `wsl -> local서버 -> 외부`로 방화벽 설정을 해주어야 함

#### 우분투 내부

```bash
# iptables 백엔드를 legacy로 변경
sudo update-alternatives --config iptables  # → 1 선택 (iptables-legacy)

# 포트 허용 (예: 56004)
sudo iptables -I INPUT 1 -p tcp --dport 56004 -j ACCEPT
```

#### windows 서버 <-> wsl 포트포워딩 (powershell 관리자 권한)

```bash
# WSL IP 확인 (WSL 내부)
hostname -I

# 포트포워딩 설정 (Windows PowerShell)
netsh interface portproxy add v4tov4 listenport=56004 listenaddress=0.0.0.0 connectport=56004 connectaddress=<WSL-IP>
```

> 제발 알아보고 해결하자