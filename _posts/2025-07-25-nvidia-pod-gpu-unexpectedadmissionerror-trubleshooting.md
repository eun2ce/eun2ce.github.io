---
title: "[GPU/K8s] NVML Insufficient Permissions 에러로 Pod GPU 할당 실패 해결"
description: "Kubernetes에서 A100 GPU(MIG/Non-MIG 혼용) 환경에서 Pod GPU 할당 시 NVML Insufficient Permissions 오류가 발생한 사례와 해결 방법 정리."
date: 2025-07-24 15:32:00 +0900
categories: [ "infrastructure", "gpu" ]
tags: [ "k8s", "nvidia", "device-plugin", "permissions", "troubleshooting" ]
pin: false
math: false
mermaid: false
---

## 문제 상황

Pod 스케줄링 과정에서 GPU 할당이 실패하고, `UnexpectedAdmissionError`가 발생.

```bash
Events:
  Warning  UnexpectedAdmissionError  18s  kubelet
  Allocate failed due to device plugin GetPreferredAllocation rpc failed with err:
  rpc error: code = Unknown desc = Unable to retrieve list of available devices:
  error creating nvml.Device 0: nvml: Insufficient Permissions, which is unexpected
```

## 원인 분석

- Kubernetes 기본 자원(CPU, 메모리)와 달리 GPU는 제조사별 플러그인을 통해 관리됨  
- NVIDIA Device Plugin이 **NVML 권한 부족**으로 디바이스 초기화 실패  
- 특히 **A100 GPU를 MIG와 Non-MIG로 혼합 사용**할 때 권한 요구 사항이 까다로움  
- 참고: [NVIDIA k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)

즉, Pod Admission 단계에서 NVML 접근 권한 부족(`Insufficient Permissions`) 때문에 자원 할당이 거부된 것.

## 해결 방법

NVIDIA Device Plugin DaemonSet에 아래 설정 추가:

| 설정                                | 역할                                       |
| --------------------------------- | ---------------------------------------- |
| `NVIDIA_MIG_MONITOR_DEVICES=all`  | GPU 및 MIG 인스턴스를 모두 모니터링하도록 지정 |
| `capabilities.add: ["SYS_ADMIN"]` | NVML을 통한 GPU 내부 정보 접근 권한 부여        |

## 최종 DaemonSet 매니페스트

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
        gpu: nvidia
      tolerations:
        - key: nvidia.com/gpu
          operator: Exists
          effect: NoSchedule
      priorityClassName: "system-node-critical"
      containers:
        - name: nvidia-device-plugin-ctr
          image: nvcr.io/nvidia/k8s-device-plugin@sha256:964847cc3fd85ead286be1d74d961f53d638cd4875af51166178b17bba90192f
          env:
            - name: FAIL_ON_INIT_ERROR
              value: "false"
            - name: MIG_STRATEGY
              value: mixed
            - name: DEVICE_DISCOVERY_STRATEGY
              value: volume-mounts
            - name: NVIDIA_MIG_MONITOR_DEVICES
              value: all
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
              add: ["SYS_ADMIN"]
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

---

## 정리

- Pod GPU 할당 실패(`UnexpectedAdmissionError`)는 대부분 **NVIDIA Device Plugin → NVML 권한 문제**  
- **SYS_ADMIN capability**를 부여하면 해결됨  
- MIG/Non-MIG 혼용 환경에서는 `NVIDIA_MIG_MONITOR_DEVICES=all` 설정이 필요  
- 공식 YAML을 기준으로, **권한과 환경변수만 최소 수정**하는 것이 가장 안전
