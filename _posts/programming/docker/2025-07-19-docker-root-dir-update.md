---
title: "[Docker] root 디렉토리 변경"
date: 2025-07-19:00 +0900
categories: [ "programming", "docker" ]
tags: ["root", "docker", "ops", "trubleshooting"]
pin: false
math: false
mermaid: false
---

클라우드를 사용하면서 스토리지가 부족한 현상이 생겼다. 찾아보니 default docker root 디렉토리는 `/var/lib/docker`로 지정되어 있는데, 거기에는 이미지, 컨테이너, 볼륨 전부 들어있기 때문에 그게 문제가 된 것이었다.

그래서 사용중인 nas쪽을 바라보도록 root경로를 변경해주기로 했다.

옮겨줄 디렉토리 생성

```
$ mkdir /nas/sys/docker
```

기존 데이터 마이그레이션

```
sudo systemctl stop docker
sudo rsync -aP /var/lib/docker/ /my/custom/docker/
```

`/etc/docker/daemon.json` 수정

```
{
  "data-root": "/nas/sys/docker"
}
```

데몬 재시작

```
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

확인

```
docker info | grep "Docker Root Dir"
```

추가로 나중에 안 것 이지만.. `systemd override`하는 방법도 좋아보였다.

```
sudo systemctl edit docker

// 파일에 아래와같이 수정
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --data-root=/my/custom/docker // 도커 데몬이 뜰 때 루트 디렉토리를 내가 만든 경로로 지정

// 적용 후 재시작
sudo systemctl daemon-reexec
sudo systemctl restart docker
```

더 안전해보인다.