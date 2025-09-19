---
title: "물리 코어 · VM 코어 · K8s CPU 이해와 성능 예측"
date: 2025-09-19 13:54:00 +0900
categories: [ "infrastructure", "k8s" ]
tags: [kubernetes, virtualization, cpu, performance]
pin: false
math: false
mermaid: false
---

쿠버네티스를 사용하다보면 자주 등장하는 개념 중 하나가 **물리 코어(Physical Core), VM 코어(vCPU), 그리고 쿠버네티스 CPU** 표기법은 비슷한데, 실제로는 완전히 다른 레벨의 추상화 단위입니다.

이 글에서는 그 차이를 정리하고 성능 예측은 어떻게 접근해야하는지 방향을 제안합니다.


## 1. 개념의 차이

| 구분                | 설명                                                                 | 특징/비고 |
|---------------------|----------------------------------------------------------------------|-----------|
| **물리 코어 (Physical Core)** | 실제 CPU 칩 안에 존재하는 연산 장치.<br>성능 = 클럭(GHz) × IPC. | 하이퍼스레딩(HT) 시 OS에는 2× 스레드(vCPU)로 보임 |
| **VM 코어 (vCPU)** | 하이퍼바이저가 물리 코어를 가상화한 단위. | Oversubscription 가능 → VM 코어 1개 ≠ 물리 코어 1개<br>성능은 클러스터 부하, 스케줄링 정책에 따라 달라짐 |
| **K8s CPU (requests/limits)** | 리눅스 cgroups로 관리되는 CPU 시간 단위 할당량. | 1 CPU = 노드 OS가 보는 1 vCPU의 100% 실행 시간<br>물리 코어 고정 점유 아님, VM 위라면 vCPU 기준으로 동작 |

> 즉, `VM 코어 1개 = 물리 코어 1개`는 성립하지 않습니다.
> 쿠버네티스의 CPU는 **시간 단위의 할당량**이지, 물리적 코어를 직접 매핑한 게 아닙니다.


## 2. "24 물리 코어 = 48 vCPU = 48 K8s CPU?"  

잘못된 접근입니다.
각 계층은 서로 다른 추상화 레벨에서 리소스를 표현하는 방식이라 **직접 환산할 수 없습니다**.  

## 3. 성능 예측은 어떻게 해야 할까?

각 계층에 맞는 방식을 사용해야 합니다.

### 1) 벤치마크로 단위 성능 측정

```bash
# sysbench 예시 (1 vCPU 테스트)
sysbench cpu --threads=1 --time=30 run
... 중략 ...
CPU speed:
    events per second:  5170.90
total number of events: 155131
```

 **1 vCPU 기준 약 5170 events/sec 처리 가능**

이 결과를 쿠버네티스에서 1 cpu 단위 성능의 기준치로 삼습니다.

### 2) pod 성능 추정


- `1 CPU` = ~5170 events/sec  
- `2 CPU` Pod = ~10,340 events/sec 기대  
- `8 CPU` Pod = ~41,360 events/sec 기대  

> 단, 실제 워크로드(멀티스레드 지원 여부, I/O 의존성)에 따라 달라집니다.  
> 따라서 벤치마크는 **참고 기준**일 뿐, 최종 성능은 부하 테스트로 확인해야 합니다.

### 3) Oversubscription 고려

- **VM 환경**: 여러 VM이 물리 코어를 공유 → 성능 변동 가능.  
- **K8s 환경**: noisy neighbor 현상 발생 가능.  

> noisy neighbor: 컨테이너가 공유 리소스(CPU, 네트워크, 디스크 I/O 등)를 과도하게 사용하여 다른 컨테이너 성능에 악영향을 주는 상황.  

**SLA가 중요하다면:**
- VM 레벨: **CPU Reservation**
- K8s 레벨: **resources.limits 설정**

## 4. 실제 하드웨어 스펙과 리소스 설정 예시

머신 스펙:
- CPU: AMD Ryzen Threadripper PRO 5955WX (16C/32T, nproc=32)  
- RAM: ~125.6 GiB  
- GPU: RTX A6000 × 2 (48GB × 2)

```yaml
# values.yaml
resources:
  requests:
    cpu: "1000m"      # 1 vCPU
    memory: "6Gi"
    nvidia.com/gpu: 1
  limits:
    cpu: "3000m"      # 3 vCPU
    memory: "28Gi"
    nvidia.com/gpu: 1
```

## 5. OOM 없이 안정적으로 운영하려면?

문제: memory limit이 너무 타이트하거나 CPU limit이 실제 처리량보다 낮으면 → **OOMKilled** 또는 **CPU throttling** 발생.

### 제안 가이드라인
1. **requests = 평균 부하, limits = 피크 부하 + 여유치**
   - CPU: requests는 평균 사용량 기준, limits는 1.5~2배 여유.
   - 메모리: requests는 최소 필요치, limits는 1.2~1.5배 여유.

2. **GPU 워크로드 고려**
   - GPU inference 시 CPU보다 메모리가 bottleneck 되는 경우 많음.
   - GPU batch size에 따라 Pod memory limit을 반드시 조정해야 함.

3. **예시 수정안**

```yaml
  resources:
    requests:
      cpu: "6000m"
      memory: "24Gi"
      nvidia.com/gpu: 1
    limits:
      cpu: "12000m"
      memory: "48Gi"
      nvidia.com/gpu: 1
```