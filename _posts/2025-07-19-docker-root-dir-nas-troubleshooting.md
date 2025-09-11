---
title: "[Docker] Root 디렉토리(data-root) 변경 및 NAS 환경 트러블슈팅"
description: "Docker Root Directory(`/var/lib/docker`)를 NAS 등 다른 디렉토리로 변경하는 방법과, 발생할 수 있는 오류(troubleshooting) 및 systemd override 활용법을 정리합니다."
date: 2025-07-19 13:00:00 +0900
categories: [ "infrastructure", "docker" ]
tags: [ "docker", "docker-root-dir", "data-root", "storage-driver", "systemd", "troubleshooting" ]
pin: false
math: false
mermaid: false
---

## 문제 상황

클라우드를 사용하면서 스토리지가 부족한 현상이 생겼습니다.  
Docker 기본 root 디렉토리(`/var/lib/docker`)에는 이미지, 컨테이너, 볼륨이 모두 저장되므로 빠르게 용량이 차는 문제가 발생합니다.  
이를 해결하기 위해 NAS 스토리지로 root 디렉토리를 변경했습니다.

---

## 경로 변경 절차

### 1. 디렉토리 생성
```bash
mkdir /nas/sys/docker
```

### 2. 기존 데이터 마이그레이션
```bash
sudo systemctl stop docker
sudo rsync -aP /var/lib/docker/ /nas/sys/docker/
```

### 3. `/etc/docker/daemon.json` 수정
```json
{
  "data-root": "/nas/sys/docker"
}
```

### 4. Docker 재시작
```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

만약 아래와 같은 오류가 발생한다면:
```bash
Job for docker.service failed because the control process exited with error code.
```

이는 NAS 등 비표준 파일시스템을 root 디렉토리로 지정할 때 발생할 수 있는 문제입니다.

---

## 트러블슈팅

### Storage Driver 변경

NAS 환경에서는 `overlay2` 드라이버 사용이 불안정할 수 있습니다.  
이 경우 `vfs` 드라이버를 지정하면 동작은 가능하지만 성능 저하가 발생합니다.

```json
{
  "data-root": "/nas/sys/docker",
  "storage-driver": "vfs"
}
```

확인:
```bash
docker info | grep "Docker Root Dir"
```

---

## systemd Override 활용

`systemd override`를 이용하면 `daemon.json` 수정 없이 root 디렉토리를 변경할 수도 있습니다.

```bash
sudo systemctl edit docker
```

편집기에서 아래 내용을 입력:
```ini
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --data-root=/my/custom/docker
```

적용 후 재시작:
```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

이 방식은 서비스 단위에서 직접 root 디렉토리를 지정하므로 더 안전하고 관리하기 용이합니다.

---

## 정리

- Docker 기본 root 디렉토리는 `/var/lib/docker`  
- NAS 등으로 옮길 경우 성능/호환성 문제 발생 가능  
- `storage-driver: vfs` 설정으로 동작은 가능하나 성능은 떨어짐  
- 장기적으로는 `systemd override` 방식이 더 안정적  
