---
title: "[NVIDIA/trubleshooting] NVML not detected: could not load NVML library: libnvidia-ml.so.1: cannot open shared object file: No such file or directory"
date: 2025-07-24 15:32:00 +0900
categories: [ "ops", "gpu" ]
tags: [ "nhncloud", "k8s", "nvidia", "troubleshooting" ]
pin: false
math: false
mermaid: false
image:
  path: assets/img/posts/2025-07-24-k3s-a100-nvidia-device-plugin-trubleshooting-2025-07-24%2015.45.49.png
---

## 구성

- OS: Ubuntu 22.04
- GPU: NVIDIA A100 (4개)
- K3s: containerd 기반
- NVIDIA Container Toolkit 설치 완료
- 드라이버 버전: `libnvidia-ml.so.575.57.08`

## 문제

- local환경에는 4개의 gpu가 있고, 2개는 mig로 나누고 남은 2개는 전체 gpu로 사용중
- `local`과 `containerd` 에서는 `nvidia-smi`으로 gpu가 조회되지만 k3s노드에서는 인식하지 못함
- gpu operator는 mig형태로 잘려있는 gpu를 섞어 사용했을 경우 설치할 수 없음 -> 그래서 찾은게 nvidia-device-plugin
- 이후 nvidia-device-plugin을 설치했지만 아래와 같은 이유로 gpu를 읽을 수 없음

```bash
root@a100-80g-4:/home/ubuntu/works/eun2ce/ops/k3s/gpu# kubectl logs -n kube-system nvidia-device-plugin-daemonset-8xmbs
I0724 06:09:00.921957       1 main.go:235] "Starting NVIDIA Device Plugin" version=<
	3c378193
	commit: 3c378193fcebf6e955f0d65bd6f2aeed099ad8ea
 >
I0724 06:09:00.922013       1 main.go:238] Starting FS watcher for /var/lib/kubelet/device-plugins
I0724 06:09:00.922061       1 main.go:245] Starting OS watcher.
I0724 06:09:00.922272       1 main.go:260] Starting Plugins.
I0724 06:09:00.922348       1 main.go:317] Loading configuration.
I0724 06:09:00.922970       1 main.go:342] Updating config with default resource matching patterns.
W0724 06:09:00.923058       1 rm.go:108] mig-strategy="mixed" is only supported with NVML
W0724 06:09:00.923072       1 rm.go:109] NVML not detected: could not load NVML library: libnvidia-ml.so.1: cannot open shared object file: No such file or directory
I0724 06:09:00.923346       1 main.go:353]
Running with config:
{
... 중략
}
I0724 06:10:10.145748       1 main.go:356] Retrieving plugins.
E0724 06:10:10.145900       1 factory.go:112] Incompatible strategy detected auto
E0724 06:10:10.145908       1 factory.go:113] If this is a GPU node, did you configure the NVIDIA Container Toolkit?
E0724 06:10:10.145914       1 factory.go:114] You can check the prerequisites at: https://github.com/NVIDIA/k8s-device-plugin#prerequisites
E0724 06:10:10.145919       1 factory.go:115] You can learn how to set the runtime at: https://github.com/NVIDIA/k8s-device-plugin#quick-start
E0724 06:10:10.145924       1 factory.go:116] If this is not a GPU node, you should set up a toleration or nodeSelector to only deploy this plugin on GPU nodes
I0724 06:10:10.145931       1 main.go:381] No devices found. Waiting indefinitely.
```

plugin 시작이 실패했고 NVIDIA Kubernetes Device Plugin에서 GPU 디바이스를 어떻게 탐지할지를 설정하는 내용을 yaml에서 바꿈

> 참고로 이 버전은 
```bash
// /dev 및 드라이버가 마운트된 경로를 직접 탐색해서 디바이스 식별
DEVICE_DISCOVERY_STRATEGY=volume-mounts
```

이후 컨테이너 내부에서 nvml 초기화 실패 (특정 라이브러리를 마운트하지 못함)

```bash
`nvml: Unknown Error`

Failed to initialize NVML: nvml: Unknown Error.
```

`nvidia-driver` 볼륨 마운트(`/usr/lib/x86_64-linux-gnu`) 제거

제거 후 host에 있는 glibc버전과 컨테이너 내부의 glibc 버전이 맞지 않는 문제 발생

```bash
relocation error: libc.so.6: symbol _dl_audit_preinit not defined in file ld-linux-x86-64.so.2
```

이후 라이브러리 hostPath 마운트 제거

확인

![](/assets/img/posts/2025-07-24-k3s-a100-nvidia-device-plugin-trubleshooting-2025-07-24%2015.45.49.png)

```bash
Capacity:
  cpu:                     48
...
  nvidia.com/gpu:          2 // gpu 2개
  nvidia.com/mig-3g.40gb:  4 // mig로 잘린 gpu
  pods:                    110
Allocatable:
  cpu:                     48
  ...
  nvidia.com/gpu:          2
  nvidia.com/mig-3g.40gb:  4
  pods:                    110
```

- k3s에서는 containerd 설정은 `/var/lib/rancher/k3s/agent/etc/containerd/config.toml`이지만 수동 수정 불가 → `config.toml.tmpl` 사용
- NVIDIA plugin은 버전과 환경변수가 민감
- 호스트 라이브러리를 컨테이너에 마운트하는 건 glibc 충돌로 이어질 수 있음
- 공식 YAML 그대로 쓰는 게 제일 빠름