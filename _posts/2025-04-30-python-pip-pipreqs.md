---
title: "[Python] pipreqs로 Conda 환경에서 requirements.txt 생성하기"
description: "Conda 환경에서 불필요한 패키지를 제외하고 실제 사용 중인 라이브러리만 추출해 requirements.txt를 생성하는 방법을 pipreqs로 정리합니다."
date: 2025-04-30 14:00:00 +0900
categories: [ "development", "language" ]
tags: [ "python", "conda", "pipreqs", "requirements.txt", "guide" ]
pin: false
math: false
mermaid: false
---

Conda 환경의 불필요한 패키지 없이 실제 사용하는 라이브러리만 뽑아냅니다.

---

# 설치

```bash
pip install pipreqs
```

# 사용법

프로젝트 루트에서:

```bash
# requirements.txt 생성
pipreqs .

# 기존 파일 덮어쓰기
pipreqs . --force
```

* --ignore 폴더명 : 스캔 제외
* --no-pin : 버전 고정 없이 패키지명만

# PyInstaller spec 파일에 반영

```python
# read_requirements() 정의
def read_requirements(path='requirements.txt'):
    reqs = []
    for line in open(path, encoding='utf-8'):
        pkg = line.strip().split('==')[0]
        if pkg and not pkg.startswith('#'):
            reqs.append(pkg)
    return reqs

# hiddenimports 에 주입
hiddenimports = read_requirements()
```
