---
title: "[Docker] root 디렉토리 변경"
date: 2025-07-19:00 13:00:00 +0900
categories: [ "ops", "docker" ]
tags: ["root", "docker", "ops", "trubleshooting"]
pin: false
math: false
mermaid: false
---

클라우드를 사용하면서 스토리지가 부족한 현상이 생겼다. 찾아보니 default docker root 디렉토리는 `/var/lib/docker`로 지정되어 있는데, 거기에는 이미지, 컨테이너, 볼륨 전부 들어있기 때문에 그게 문제가 된 것이었다.

그래서 사용중인 nas쪽을 바라보도록 root경로를 변경해주기로 했다.

옮겨줄 디렉토리 생성

```bash
$ mkdir /nas/sys/docker
```

기존 데이터 마이그레이션

```bash
sudo systemctl stop docker
sudo rsync -aP /var/lib/docker/ /my/custom/docker/
          6,143 100%   70.58kB/s    0:00:00 (xfr#261367, to-chk=10/306329)
overlay2/zp4k5x5eawdxrr662qht6ansz/diff/app/utils/__pycache__/
overlay2/zp4k5x5eawdxrr662qht6ansz/diff/app/utils/__pycache__/__init__.cpython-310.pyc
            604 100%    6.94kB/s    0:00:00 (xfr#261368, to-chk=8/306329)
overlay2/zp4k5x5eawdxrr662qht6ansz/diff/app/utils/__pycache__/combine_image.cpython-310.pyc
            912 100%   10.48kB/s    0:00:00 (xfr#261369, to-chk=7/306329)
overlay2/zp4k5x5eawdxrr662qht6ansz/diff/app/utils/__pycache__/logger.cpython-310.pyc
          3,769 100%   43.30kB/s    0:00:00 (xfr#261370, to-chk=6/306329)
overlay2/zp4k5x5eawdxrr662qht6ansz/diff/app/utils/__pycache__/utils.cpython-310.pyc
          7,191 100%   82.62kB/s    0:00:00 (xfr#261371, to-chk=5/306329)
overlay2/zp4k5x5eawdxrr662qht6ansz/work/
overlay2/zp4k5x5eawdxrr662qht6ansz/work/work/
overlay2/zp4k5x5eawdxrr662qht6ansz/work/work/#a9a
plugins/
plugins/tmp/
runtimes/
swarm/
tmp/
volumes/
volumes/backingFsBlockDev
volumes/metadata.db
         32,768 100%  372.09kB/s    0:00:00 (xfr#261372, to-chk=0/306329)
$
```

`/etc/docker/daemon.json` 수정

```bash
{
  "data-root": "/nas/sys/docker"
}
```

데몬 재시작

```bash
sudo systemctl daemon-reexec
sudo systemctl restart docker
Job for docker.service failed because the control process exited with error code.
See "systemctl status docker.service" and "journalctl -xeu docker.service" for details.
```

위와같은 오류는 nas등과같은 비표준 파일시스템을 docker root dir로 지정한 경우 나타난다.
`"storage-driver": "vfs"`설정을 추가하면 속도는 느리지만 사용은 가능하다.

```bash
{
    "data-root": "/nas/sys/docker",
    "storage-driver": "vfs",
}
```


확인

```bash
docker info | grep "Docker Root Dir"
```

추가로 나중에 안 것 이지만.. `systemd override`하는 방법도 좋아보였다.

```bash
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