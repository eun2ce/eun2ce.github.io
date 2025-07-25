---
title: "[NVIDIA/trubleshooting] Allocate failed due to device plugin GetPreferredAllocation rpc failed with err: rpc error: code = Unknown desc = Unable to retrieve list of available devices: error creating nvml.Device 0: nvml: Insufficient Permissions, which is unexpected"
date: 2025-07-24 15:32:00 +0900
categories: [ "ops", "gpu" ]
tags: [ "nhncloud", "k8s", "nvidia", "troubleshooting" ]
pin: false
math: false
mermaid: false
---

k8s pod 에 gpu를 할당하는 과정에서 발생한 UnexpectedAdmissionError 를 해결합니다.

```bash
Events:
  Type     Reason                    Age   From                       Message
  ----     ------                    ----  ----                       -------
  Normal   Scheduled                 18s   jupyterhub-user-scheduler  Successfully assigned jupyter/jupyter-infofla to a100-80g-4
  Warning  UnexpectedAdmissionError  18s   kubelet                    Allocate failed due to device plugin GetPreferredAllocation rpc failed with err: rpc error: code = Unknown desc = Unable to retrieve list of available devices: error creating nvml.Device 0: nvml: Insufficient Permissions, which is unexpected
```

## 분석

- UnexpectedAdmissionError는 nivida gpu 관련된 에러가 발생했을 때 볼 수 있는 상태 메시지
- kubectl을 통해 별다른 로그 없음

즉, k8s에러가 아니라 k8s에서 nvidia gpu를 알기위해(?) 사용하는 nvidia extension관련 에러일 가능성 높음

> CPU, Memory와 별개로 GPU는 종류가 여러개가 있다보니, kubernetes에서 자원 할당 정책을 적용해주기 위해서는 제조사 별로 extension을 별도로 사용해줘야한다고 함

 nvml.Device 의 권한 에러 발생으로 확인할 수 있는데, device관련 사항은 nvidia extension 중 nvidia-device-plugin이 관리함

 [https://github.com/NVIDIA/k8s-device-plugin](https://github.com/NVIDIA/k8s-device-plugin)

 - 클러스터 노드마다 gpu 개수를 k8s에 안내
 - gpu 상태 추적
 - k8s 클러스터 내 컨테이너에서 gpu를 사용할 수 있도록 기능 제공

 > 관련 에러 메시지 [https://github.com/NVIDIA/k8s-device-plugin/issues/260](https://github.com/NVIDIA/k8s-device-plugin/issues/260)
 > [https://equus3144.medium.com/nvidia-ml-py%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-kubernetes%EC%97%90-%EB%B0%B0%ED%8F%AC%EB%90%98%EC%96%B4-%EC%9E%88%EB%8A%94-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%EC%97%90%EC%84%9C-mig-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%82%AC%EC%9A%A9%EB%9F%89-%EC%B2%B4%ED%81%AC%ED%95%98%EA%B8%B0-c73a70d700fa](https://equus3144.medium.com/nvidia-ml-py%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%84%9C-kubernetes%EC%97%90-%EB%B0%B0%ED%8F%AC%EB%90%98%EC%96%B4-%EC%9E%88%EB%8A%94-%EC%9D%B8%EC%8A%A4%ED%84%B4%EC%8A%A4%EC%97%90%EC%84%9C-mig-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%82%AC%EC%9A%A9%EB%9F%89-%EC%B2%B4%ED%81%AC%ED%95%98%EA%B8%B0-c73a70d700fa)

 즉, A100 GPU를 분할 사용 하지 않는 경우를 섞어 사용할 때 문제가 발생


필자는 아래 두 옵션을 추가해서 문제 해결

| 설정                                | 역할                           |
| --------------------------------- | ---------------------------- |
| `NVIDIA_MIG_MONITOR_DEVICES=all`  | 어떤 GPU (MIG 인스턴스)를 모니터링할지 지정 |
| `capabilities.add: ["SYS_ADMIN"]` | 그 GPU의 내부 정보에 접근할 권한을 부여     |


```yaml
# nvidia-device-plugin.yml
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
        - image: nvcr.io/nvidia/k8s-device-plugin@sha256:964847cc3fd85ead286be1d74d961f53d638cd4875af51166178b17bba90192f
          name: nvidia-device-plugin-ctr
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
              add: ["SYS_ADMIN"]
                #drop: ["ALL"]
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