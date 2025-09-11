---
title: "[클라우드] 재부팅 시에도 유지되는 블록 스토리지 마운트 설정"
description: "클라우드 환경에서 블록 스토리지를 연결하고, 재부팅 이후에도 마운트가 유지되도록 fstab 설정 및 자동화 스크립트를 사용하는 방법을 정리합니다."
date: 2025-07-22 10:24:00 +0900
categories: [ "infrastructure", "cloud" ]
tags: [ "nhncloud", "storage", "fstab", "mount", "troubleshooting" ]
pin: false
math: false
mermaid: false
---

## 문제 상황

클라우드 인스턴스 기본 스토리지가 부족해 추가 블록 스토리지를 연결했습니다.  
하지만 `mount` 명령어로 마운트하면 재부팅 시 해제되므로, 이를 영구적으로 유지할 필요가 있습니다.

```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1       194G  172G   23G  89% /
```

---

## 블록 스토리지 연결 및 파티션 생성

스토리지 추가 후 확인하면 `vdb` 장치가 나타납니다.

```bash
$ lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   200G  0 disk
└─vda1  252:1    0 199.9G  0 part /
vdb     252:16   0  1000G  0 disk
```

### 파티션 생성
```bash
fdisk /dev/vdb
# → n (새 파티션), p (primary), w (저장)
```

### 포맷
```bash
mkfs -t xfs /dev/vdb1
```

---

## fstab 등록 (재부팅 후에도 유지)

`blkid` 로 UUID 확인:
```bash
$ blkid /dev/vdb1
/dev/vdb1: UUID="30b39801-xxxx-xxxx-xxxx-33f40c21a64a" TYPE="xfs"
```

`/etc/fstab` 에 등록:
```bash
UUID=30b39801-xxxx-xxxx-xxxx-33f40c21a64a /mnt/vdb xfs defaults,nodev,noatime,nofail 1 2
```

반영:
```bash
mkdir -p /mnt/vdb
mount -a
```

확인:
```bash
df -h | grep vdb
/dev/vdb1      1000G  7.1G  993G   1% /mnt/vdb
```

### fstab 옵션 설명
- `nofail` : 마운트 실패 시에도 부팅이 계속됨  
- `nodev` : 장치 파일 생성 방지  
- `noatime` : 파일 접근 시간 기록 생략 (성능 개선)  

---

## 자동화 스크립트

여러 블록 디스크를 자동으로 초기화 및 마운트하려면 스크립트를 사용할 수 있습니다.

### CentOS 7 (XFS)
```bash
#!/bin/bash
DEVICES=$(lsblk -d -o name,type | grep disk | grep -v vda | awk '{print $1}')
for DEVICE_NAME in ${DEVICES[@]}; do
   MOUNT_DIR="/mnt/$DEVICE_NAME"
   FS_TYPE=xfs
   mkdir -p "$MOUNT_DIR"
   echo -e "n
p
1


w" | fdisk "/dev/$DEVICE_NAME"
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE "$PART_NAME" > /dev/null
   UUID=$(blkid "$PART_NAME" -o export | grep "^UUID=" | cut -d'=' -f 2)
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab
done
mount -a
```

### Ubuntu / CentOS 6 (ext4)
```bash
#!/bin/bash
DEVICES=$(lsblk -d -o name,type | grep disk | grep -v vda | awk '{print $1}')
for DEVICE_NAME in ${DEVICES[@]}; do
   MOUNT_DIR="/mnt/$DEVICE_NAME"
   FS_TYPE=ext4
   mkdir -p "$MOUNT_DIR"
   echo -e "n
p
1


w" | fdisk "/dev/$DEVICE_NAME"
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE "$PART_NAME" > /dev/null
   UUID=$(blkid "$PART_NAME" -o export | grep "^UUID=" | cut -d'=' -f 2)
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab
done
mount -a
```

---

## 정리

- `mount` 만으로는 재부팅 시 마운트가 유지되지 않음  
- `fstab`에 UUID 기반으로 등록하면 안전하게 유지 가능  
- `nofail`, `noatime` 같은 옵션을 통해 안정성과 성능도 고려 가능  
- 스크립트로 여러 디스크를 한 번에 초기화/마운트하면 편리함  
