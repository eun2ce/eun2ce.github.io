---
title: "[클라우드] 재부팅 시에도 유지되는 블록 스토리지 마운트 설정"
date: 2025-07-22 10:24:00 +0900
categories: [ "ops", "cloud" ]
tags: [ "nhncloud", "storage", "troubleshooting" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-07-22-cloud-block-storage-mount-2025-07-22-10-53-17.webp
---

클라우드 기본 사용량이 적어 스토리지 89%까지 차서 문제가 되었다.

```
(base) ubuntu@a100-80g-4:~/works/eun2ce/ops$ df -h
Filesystem                                                      Size  Used Avail Use% Mounted on
tmpfs                                                            89G  1.2M   89G   1% /run
/dev/vda1                                                       194G  172G   23G  89% /
```

아래와 같이 블록 스토리지를 생성 후 연결

![](/assets/img/posts/2025-07-22-cloud-block-storage-mount-2025-07-22-10-53-17.webp)

이후 인스턴스에 접근하면 아래와같이 루트 블록스토리지와 추가 블록스토리지인 \`vdb\`를 볼 수 있다.

```
root@a100-80g-4:/home/ubuntu# lsblk
NAME    MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
vda     252:0    0   200G  0 disk
├─vda1  252:1    0 199.9G  0 part /
├─vda14 252:14   0     4M  0 part
└─vda15 252:15   0   106M  0 part /boot/efi
vdb     252:16   0  1000G  0 disk
```

파티션 생성

```
root@a100-80g-4:/home/ubuntu# fdisk /dev/vdb // vdb에 파티션 생성

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x6c11138e.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-2097151999, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151999, default 2097151999):

Created a new partition 1 of type 'Linux' and of size 1000 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

-   `primary`는 최대 4개까지만 생성 가능
-   5개 이상이 필요할 경우:
    -   `primary` 3개 + `extended` 1개 + 그 안에 여러 개의 `logical`
-   `logical`은 반드시 `extended` 안에만 존재할 수 있음

파티션 포맷

```
root@a100-80g-4:/home/ubuntu# lsblk /dev/vdb
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vdb    252:16   0 1000G  0 disk
└─vdb1 252:17   0 1000G  0 part
root@a100-80g-4:/home/ubuntu# mkfs -t xfs /dev/vdb1
meta-data=/dev/vdb1              isize=512    agcount=4, agsize=65535936 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0 inobtcount=0
data     =                       bsize=4096   blocks=262143744, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=127999, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
root@a100-80g-4:/home/ubuntu#
```

블록 스토리지 마운트

> `mount` 명령어로 블록 스토리지를 마운트 할 수도 있으나 인스턴스가 재부팅되면 마운트 해제(unmount)되므로 아래와 같은 방법 추천

```
root@a100-80g-4:/home/ubuntu# cat /etc/fstab
LABEL=cloudimg-rootfs    /     ext4    discard,errors=remount-ro    0 1
LABEL=UEFI    /boot/efi    vfat    umask=0077    0 1

192.168.0.8:/GJ_SHARE_FS8/29a...06724c4 /nas nfs nfsvers=3,auto,nofail,noatime 0 0 // 이미 하나가 마운트 되어있음
root@a100-80g-4:/home/ubuntu# blkid /dev/vdb1 // UUID 조회
/dev/vdb1: UUID="30b39801........33f40c21a64a" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="6c11138e-01"
root@a100-80g-4:/home/ubuntu# mkdir -p /mnt/vdb // 마운트 될 디렉토리 생성
root@a100-80g-4:/home/ubuntu# echo "UUID=30b39801-a9a2-43c3-a3e1-33f40c21a64a /mnt/vdb xfs defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

// mount 반영
root@a100-80g-4:/home/ubuntu# mount -a
root@a100-80g-4:/home/ubuntu# df -h
Filesystem                                                      Size  Used Avail Use% Mounted on
tmpfs                                                            89G  1.2M   89G   1% /run
/dev/vda1                                                       194G  179G   16G  93% /
tmpfs                                                           443G   56K  443G   1% /dev/shm
tmpfs                                                           5.0M     0  5.0M   0% /run/lock
tmpfs                                                           4.0M     0  4.0M   0% /sys/fs/cgroup
/dev/vda15                                                      105M  6.1M   99M   6% /boot/efi
192.168.0.8:/GJ_SHARE_FS8/29a360d3-2dfc-4d32-818e-b475406724c4  4.0T   40G  3.9T   1% /nas
tmpfs                                                            89G   36K   89G   1% /run/user/1000
overlay                                                         194G  179G   16G  93% /var/lib/docker/overlay2/c35015e74668b679d2bb103a85763845b4357dfe3cfdd9244e05933e2a5ebba4/merged
/dev/vdb1                                                      1000G  7.1G  993G   1% /mnt/vdb // 반영된 것 확인
root@a100-80g-4:/home/ubuntu#
```

-   볼륨 마운트에 실패하더라도 부팅이 될 수 있도록 `nofail` 옵션을 추가하는 것을 권장
-   nodev: 이 파티션에 장치 파일 생성 방지, noatime: 접근시간 기록 안 해서 속도 향상

추가로 위 일을 한번에 처리하는 스크립트

CentOS 7

```

#!/bin/bash

DEVICES=$(lsblk -d -o name,type | grep disk | grep -v vda | awk '{print $1}')

for DEVICE_NAME in ${DEVICES[@]}
do
   MOUNT_DIR="/mnt/$DEVICE_NAME"
   FS_TYPE=xfs

   mkdir -p "$MOUNT_DIR"

   echo -e "n\np\n1\n\n\nw" | fdisk "/dev/$DEVICE_NAME"
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE "$PART_NAME" > /dev/null

   UUID=$(blkid "$PART_NAME" -o export | grep "^UUID=" | cut -d'=' -f 2)
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

done

mount -a
```

CentOS6, Debian, Ubuntu는 기본 파일 시스템이 ext4

```
#!/bin/bash

DEVICES=$(lsblk -d -o name,type | grep disk | grep -v vda | awk '{print $1}')

for DEVICE_NAME in ${DEVICES[@]}
do
   MOUNT_DIR="/mnt/$DEVICE_NAME"
   FS_TYPE=ext4

   mkdir -p "$MOUNT_DIR"

   echo -e "n\np\n1\n\n\nw" | fdisk "/dev/$DEVICE_NAME"
   PART_NAME="/dev/${DEVICE_NAME}1"
   mkfs -t $FS_TYPE "$PART_NAME" > /dev/null

   UUID=$(blkid "$PART_NAME" -o export | grep "^UUID=" | cut -d'=' -f 2)
   echo "UUID=$UUID $MOUNT_DIR $FS_TYPE defaults,nodev,noatime,nofail 1 2" >> /etc/fstab

done

mount -a
```