---
title: "[GPU] FlashAttention2 설치 오류로 Qwen 모델 로딩 실패 해결"
description: "FlashAttention2 설치 문제로 인해 Qwen 모델 로딩에 실패하는 오류를 CUDA 버전 업그레이드(11.5 → 12.6)로 해결한 과정을 정리합니다."
date: 2025-07-20 12:39:00 +0900
categories: [ "infrastructure", "gpu" ]
tags: [ "flashattention2", "qwen", "huggingface", "cuda", "a100", "installation-error", "troubleshooting" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-07-20-etc-flashattention2-cuda-trubleshooting-img.webp
  alt: "FlashAttention2 설치 오류로 Qwen 모델 로딩 실패"
---

## 문제 현상

Qwen 모델을 로딩하는 과정에서 다음과 같은 오류가 발생했습니다.

```bash
ImportError: FlashAttention2 has been toggled on, but it cannot be used due to the following error: the package flash_attn seems to be not installed. Please refer to the documentation of https://huggingface.co/docs/transformers/perf_infer_gpu_one#flashattention-2 to install Flash Attention 2.
```

---

## 원인 분석

1. **CUDA 버전 불일치**  
   - 서버에는 CUDA 11.5가 설치되어 있었으나, FlashAttention2는 **CUDA 12.6 이상**을 요구합니다.

   ```bash
   $ nvcc -V
   nvcc: NVIDIA (R) Cuda compiler driver
   Cuda compilation tools, release 11.5, V11.5.119
   ```

2. **Transformers의 자동 활성화**  
   - 최신 `transformers` 라이브러리는 FlashAttention2가 사용 가능한 환경일 경우 자동으로 기능을 켭니다.  
   - CUDA 버전이 낮으면 ImportError가 발생합니다.

---

## 해결 방법

### CUDA 12.6 설치 (Ubuntu 22.04 기준)

NVIDIA 공식 저장소를 추가하고 CUDA 12.6을 설치합니다.

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-6
```

설치 완료 후 버전을 확인합니다.

```bash
$ nvcc -V
Cuda compilation tools, release 12.6, V12.6.XXX
```

---

## 정리

- FlashAttention2는 CUDA 12.6 이상 환경이 필요합니다.  
- CUDA 버전이 낮으면 `transformers`에서 자동 활성화된 FlashAttention2가 ImportError를 발생시킵니다.  
- CUDA를 업그레이드하면 Qwen 모델 로딩이 정상적으로 동작합니다.

---

![img](/assets/img/posts/2025-07-20-etc-flashattention2-cuda-trubleshooting-img.webp)
