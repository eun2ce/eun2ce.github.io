---
title: "[GPU/K8s] NVML not detected & libnvidia-ml.so.1 오류로 NVIDIA Device Plugin이 GPU를 인식하지 못할 때"
description: "K3s(containerd) 환경에서 NVML 로드 실패(libnvidia-ml.so.1)로 NVIDIA Kubernetes Device Plugin이 기동되지 않는 문제를, 호스트 라이브러리 마운트 제거 및 volume-mounts 전략으로 해결한 사례 정리."
date: 2025-07-24 15:32:00 +0900
categories: [ "infrastructure", "gpu" ]
tags: [ "k8s", "k3s", "nvidia", "device-plugin", "mig", "troubleshooting" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-07-24-k3s-a100-nvidia-device-plugin-trubleshooting-2025-07-24%2015.45.49.webp
  alt: "K3s에서 NVIDIA Device Plugin 트러블슈팅"
---

## 구성

- **OS**: Ubuntu 22.04
- **GPU**: NVIDIA A100 (4장, 일부 MIG 분할)
- **K3s**: containerd 기반
- **NVIDIA Container Toolkit**: 설치 완료
- **드라이버**: `libnvidia-ml.so.575.57.08`

## 증상

- 호스트/컨테이너에서는 `nvidia-smi` 동작하나, **k3s 노드 리소스**로는 GPU가 보이지 않음  
- GPU Operator는 *MIG와 non‑MIG* 혼용 노드에서 구성 난이도가 높아 **NVIDIA Device Plugin**으로 전환  
- Device Plugin 기동 로그에서 **NVML 로드 실패**로 플러그인이 디바이스를 찾지 못함

```bash
W0724 ... mig-strategy="mixed" is only supported with NVML
W0724 ... NVML not detected: could not load NVML library: libnvidia-ml.so.1: cannot open shared object file: No such file or directory
E0724 ... Incompatible strategy detected auto
...
I0724 ... No devices found. Waiting indefinitely.
```

## 원인 정리

1. **NVML 동적 라이브러리 탐색 실패**  
   - 컨테이너 안에서 `libnvidia-ml.so.1` 로딩이 실패 → NVML 초기화 불가
2. **호스트 라이브러리 hostPath 마운트의 부작용**  
   - `/usr/lib/x86_64-linux-gnu` 등을 **직접 마운트**하면 컨테이너의 **glibc**와 **호스트 glibc**가 **불일치**  
   - 결과적으로 아래와 같은 **relocation error** 발생
     ```bash
     relocation error: libc.so.6: symbol _dl_audit_preinit not defined ...
     ```
3. **MIG 전략 설정 충돌**  
   - `MIG_STRATEGY=mixed`는 **NVML 기반** 탐지에서만 유효 → NVML 미탑재 시 경고

## 해결 요약

- **라이브러리 hostPath 마운트 제거** (특히 `/usr/lib/x86_64-linux-gnu` 등)  
- 디바이스 플러그인의 탐지 전략을 **`DEVICE_DISCOVERY_STRATEGY=volume-mounts`** 로 사용  
  - `/dev` 등 **디바이스 파일만 마운트**하여 탐지
- `MIG_STRATEGY`는 **`none`** 으로 둬서 NVML 의존 경고 제거  
  - volume‑mounts 전략에서는 MIG 디바이스도 `/dev` 경유로 자동 인식됨
- **RuntimeClass `nvidia`** 및 **노드 라벨/톨러레이션**을 활용해 GPU 노드에만 배포

## 최종 동작 확인

- 노드 리소스에 **동시에** 노출됨
  ```bash
  nvidia.com/gpu:          2      # 전체 GPU 2장
  nvidia.com/mig-3g.40gb:  4      # MIG 인스턴스 4개
  ```

---

## 최종 DaemonSet 매니페스트 (수정본)

> 사전 조건
> 1) RuntimeClass `nvidia`가 존재해야 합니다. (`nvidia-container-runtime` 연동)  
> 2) GPU 노드에 `gpu=nvidia` 라벨이 있어야 합니다.  
> 3) 기본 제공 YAML을 **가능한 수정 최소화**로 사용하는 것을 권장합니다.

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        name: nvidia-device-plugin-ds
    spec:
      runtimeClassName: nvidia
      nodeSelector:
        gpu: "nvidia"         # GPU 노드만 대상
      tolerations:
        - key: "nvidia.com/gpu"
          operator: "Exists"
          effect: "NoSchedule"
      priorityClassName: "system-node-critical"
      containers:
        - name: nvidia-device-plugin-ctr
          image: nvcr.io/nvidia/k8s-device-plugin@sha256:964847cc3fd85ead286be1d74d961f53d638cd4875af51166178b17bba90192f
          env:
            - name: FAIL_ON_INIT_ERROR
              value: "false"
            # NVML 없이 /dev 스캔으로 탐지
            - name: DEVICE_DISCOVERY_STRATEGY
              value: "volume-mounts"
            # NVML 의존 경고 방지 (mixed/none 중 NVML 미사용이면 none 권장)
            - name: MIG_STRATEGY
              value: "none"
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: device-plugin
              mountPath: /var/lib/kubelet/device-plugins
            - name: nvidia-dev
              mountPath: /dev
              readOnly: true
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
        - name: nvidia-dev
          hostPath:
            path: /dev
```

### 왜 이렇게 바꿨나? (모순/위험 요소 교정)

- **(교정)** `MIG_STRATEGY=mixed`는 **NVML 필요** → volume‑mounts만 쓸 때는 `none`으로 맞춤  
- **(교정)** 호스트의 **glibc/드라이버 라이브러리**를 컨테이너에 통째로 마운트하는 패턴 제거 → **ABI 충돌** 방지  
- **(유지)** `/dev` 마운트로 디바이스만 노출 → 컨테이너 내부 libc와 충돌 없음  
- **(유지)** RuntimeClass `nvidia` 사용 → nvidia-container-runtime 경유 실행

---

## 체크리스트

- [ ] `kubectl get runtimeclass` 에 `nvidia` 존재  
- [ ] `kubectl label nodes <GPU노드> gpu=nvidia` 적용  
- [ ] Device Plugin Pod 로그에 **NVML 경고/에러 없음**  
- [ ] 노드 리소스에 `nvidia.com/gpu`, `nvidia.com/mig-*` 노출 확인  
- [ ] 워크로드 스펙에서 `resources.limits`에 위 리소스 키 사용

## 참고 명령

```bash
# Device Plugin 로그
kubectl -n kube-system logs -l name=nvidia-device-plugin-ds

# 노드 리소스 확인
kubectl describe node <node-name> | sed -n '/Capacity:/,/Allocatable:/p'

# MIG 상태
nvidia-smi -L
nvidia-smi mig -lgi -lci
```

## 결론

- **NVML 의존 제거 + /dev 기반 탐지(volume‑mounts)** 로 단순하고 견고하게 구성  
- **호스트 라이브러리 마운트는 지양** (glibc 충돌 위험)  
- **공식 YAML 최대한 유지**하면서 필요한 최소 변경만 적용하는 것이 가장 빠르게 안정화됩니다.
