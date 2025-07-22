---
title: "[Docker] Docker를 활용한 macOS → Windows 실행파일(.exe) 빌드 자동화"
date: 2025-04-08 21:47:00 +0900
categories: [ "ops", "docker" ]
tags: ["pyinstaller", "docker", "cross-compilation", "windows", "exe"]
pin: false
math: false
mermaid: false
---

> 예제코드가 필요하다면 [https://github.com/eun2ce/pyinstaller-cross-build](https://github.com/eun2ce/pyinstaller-cross-build)을 참고하세요.
{: .prompt-info }

Docker를 활용해 macOS 환경에서 Windows 실행파일(.exe)을 생성할 수 있는 방법을 정리합니다.  
빌드 과정을 자동화하고, `requirements.txt` 및 `.spec` 파일을 포함한 구조를 통해  
재사용 가능한 형태로 구성합니다.

## 개요

- Python 코드(`hello.py`)를 기준으로
- `requirements.txt`, `hello.spec`을 기반으로 의존성과 설정을 관리하고
- Docker 환경에서 cross-build를 통해 `.exe` 파일을 생성합니다.

이를 통해 macOS에서도 Windows 타겟 실행파일을 안정적으로 생성할 수 있도록 합니다.

## 프로젝트 구조

```yaml
pyinstaller-example/
 ├── hello.py
 ├── requirements.txt
 ├── hello.spec
 └── build.sh
```

### hello.py

기본 출력 외에 HTTP 요청을 통해 외부 라이브러리 동작도 함께 테스트합니다.

```python
import requests

print("Hello from PyInstaller!")
print("Fetching something from the internet...")
res = requests.get("https://httpbin.org/get")
print(res.status_code)
```

### requirements.txt

빌드시 필요한 의존성은 별도 파일로 관리합니다.  
여기서는 단순히 requests 패키지만 사용합니다.

```text
requests
```

### hello.spec

빌드 대상 파일과 실행 형태를 명시합니다.
필요 시 hidden import, data 파일 추가도 이 파일을 통해 관리합니다.

```python
# -*- mode: python ; coding: utf-8 -*-
block_cipher = None

a = Analysis(
    ['hello.py'],
    pathex=[],
    binaries=[],
    datas=[],
    hiddenimports=[],
    hookspath=[],
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
)
pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    [],
    exclude_binaries=True,
    name='hello',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    console=True,
)
coll = COLLECT(
    exe,
    a.binaries,
    a.zipfiles,
    a.datas,
    strip=False,
    upx=True,
    name='hello'
)
```

### build.sh

```bash
#!/bin/bash
docker run --rm -v "$PWD:/src" cdrx/pyinstaller-windows \
    "sh -c 'pip install -r requirements.txt && pyinstaller hello.spec'"
```

빌드 자동화를 위한 스크립트입니다.
실행 권한 부여 후 사용합니다:

```bash
chmod +x build.sh
```

## 실행 방법

```bash
./build.sh
```

실행 시 dist/hello.exe 파일이 생성됩니다.

## 실행 예시

```csharp
Hello from PyInstaller!
Fetching something from the internet...
200
```

## 정리

* macOS 환경에서 Windows 실행파일 빌드는 PyInstaller 단독으로는 어렵습니다.
* cdrx/pyinstaller-windows Docker 이미지를 활용하면 cross-build가 가능하며, CI/CD 환경에서도 활용할 수 있습니다. 
* .spec 파일과 의존성 파일을 분리해 관리하면 유지보수에 유리합니다.

반복되는 작업은 자동화하고, 빌드 환경은 컨테이너로 고정하여
재현 가능하고 안정적인 개발 프로세스를 만드는 데 목적이 있습니다.
