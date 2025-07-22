---
title: "[infra/troubleshooting] FlashAttention2 설치 오류로 인한 Qwen 모델 로딩 실패 해결"
date: 2025-07-20 12:39:00 +0900
categories: [ "ops", "gpu" ]
tags: [ "flashattention2", "qwen", "huggingface", "cuda", "a100", "troubleshooting"]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-07-20-etc-flashattention2-cuda-trubleshooting-img.png
---

신규 서버에 서빙용 모델을 올리던 중 아래와 같은 오류 발생.

```bash
ImportError: FlashAttention2 has been toggled on, but it cannot be used due to the following error: the package flash_attn seems to be not installed. Please refer to the documentation of https://huggingface.co/docs/transformers/perf_infer_gpu_one#flashattention-2 to install Flash Attention 2.
```

## 원인

1. 로컬에 설치된 cuda버전이 클라우드에서 제공하기로 한 베이스 이미지 버전보다 낮았음.

```bash
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2021 NVIDIA Corporation
Built on Thu_Nov_18_09:45:30_PST_2021
Cuda compilation tools, release 11.5, V11.5.119 // 해당 패키지는 12.6 이상 요구
Build cuda_11.5.r11.5/compiler.30672275_0
```

2. `transformers`에서 FlashAttention2 기능이 자동으로 활성화됨.

## 해결

cuda 12.6 설치

https://developer.nvidia.com/cuda-12-6-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=22.04&target_type=deb_network 에서 Linux-x86_64-Ubuntu-22.04-deb(network) 고른 후 

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb // 이렇게도 가능
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-6
```

![img](/assets/img/posts/2025-07-20-etc-flashattention2-cuda-trubleshooting-img.png)