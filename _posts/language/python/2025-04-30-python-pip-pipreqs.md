---
title: "[python] Conda 환경의 불필요한 패키지 없이 요구사항 파일 생성하기: pipreqs 활용 가이드"
date: 2025-02-07 13:00:00 +0900
categories: [ "language", "python" ]
tags: [ "algorithm", "function", "python", "사용법", "프로그래밍" ]
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
