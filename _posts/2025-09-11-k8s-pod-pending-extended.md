---
title: "Kubernetes Pod Pending 이슈 정리"
description: "스케일을 내렸다가 다시 올렸을 때 서비스가 안 올라오고 pod가 pending 상태로 남는 경우에 대한 원인, 이벤트 메시지 정리"
date: 2025-09-11 11:06:00 +0900
categories: [ "infrastructure", "k8s" ]
tags: [ "kubernetes", "pod", "pending" ]
pin: false
math: false
mermaid: false
---


스케일을 내렸다가 다시 올렸을 때 **서비스가 안 올라오고 Pod가 Pending 상태**로 남는 경우가 자주 발생합니다.  
이때 발생 가능한 원인과 `kubectl describe` 시 이벤트 메시지 예시를 정리했습니다.

---

## 주요 원인

1. **리소스 부족**
   - Pod의 `requests.cpu`, `requests.memory`, `nvidia.com/gpu`를 만족할 수 있는 노드가 없음.

2. **스토리지 대기 (PVC/PV 문제)**
   - `PersistentVolumeClaim`이 `Pending` 상태.  
   - `WaitForFirstConsumer` 스토리지클래스 사용 시, 특정 노드/존 조건이 맞지 않아 볼륨 바인딩이 지연.

3. **노드 셀렉터/어피니티/테인트 문제**
   - `nodeSelector`, `nodeAffinity`, `podAffinity/antiAffinity`, `taints/tolerations` 조건 불일치.

4. **리소스쿼터/리미트레인지 초과**
   - 네임스페이스 `ResourceQuota`를 초과했거나 `LimitRange` 규칙 위반.

5. **이미지 풀/네트워크 문제**
   - 레지스트리 접근 불가, 인증 오류, 이미지 없음 등.

6. **네임스페이스/네트워크 정책 문제**
   - Pod 네트워크 정책(NetworkPolicy)에서 허용되지 않아 준비가 지연되거나 CNI 플러그인 문제.

7. **노드 상태 이상**
   - 노드가 `NotReady` 상태거나, `DiskPressure`, `MemoryPressure` 같은 컨디션으로 스케줄 불가.

8. **PodSecurityPolicy / PodSecurity Admission**
   - Pod 스펙이 보안 정책에 어긋나서 스케줄링 불가.

---

## kubectl 이벤트에서 확인 가능한 메시지 예시

Pod가 Pending일 때 `kubectl describe pod <POD_NAME>` 실행 시 **Events** 섹션에서 다음과 같은 메시지가 나타납니다.

- 리소스 부족:
  ```text
  0/5 nodes are available: 5 Insufficient memory.
  0/3 nodes are available: 3 Insufficient nvidia.com/gpu.
  ```

- PVC 바인딩 실패:
  ```text
  waiting for a volume to be created, or waiting for volume to be bound to claim
  ```

- 노드 어피니티 불일치:
  ```text
  0/4 nodes are available: 4 node(s) didn't match node selector.
  ```

- 테인트 문제:
  ```text
  0/2 nodes are available: 2 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.
  ```

- 리소스쿼터 초과:
  ```text
  pods "mypod" is forbidden: exceeded quota: compute-resources,
  requested: limits.memory=2Gi, used: limits.memory=4Gi, limited: 5Gi
  ```

- 이미지 풀 실패:
  ```text
  Failed to pull image "myrepo/myimage:tag": rpc error: code = Unknown desc = Error response from daemon: manifest not found
  ```

- 노드 상태 이상:
  ```text
  0/4 nodes are available: 1 node(s) were not ready, 3 node(s) had disk pressure.
  ```

- PodSecurityPolicy/보안 정책 위반:
  ```text
  pods "mypod" is forbidden: violates PodSecurity "restricted:latest": 
  hostPath volumes (volume: myvol) are not allowed
  ```

---

## 진단 순서 (kubectl)

```bash
# Pod 상세 상태 확인 (Events 포함)
kubectl describe pod <POD_NAME> -n <NAMESPACE>

# 이벤트 타임라인
kubectl get events -n <NAMESPACE> --sort-by=.metadata.creationTimestamp

# PVC 상태
kubectl get pvc -n <NAMESPACE>
kubectl describe pvc <PVC_NAME> -n <NAMESPACE>

# 노드 상태/테인트
kubectl get nodes -o wide
kubectl describe nodes | egrep -i 'Allocatable|Allocated resources|Taints|Conditions'

# 네임스페이스 자원 제한 확인
kubectl get resourcequota -n <NAMESPACE> -o wide
kubectl get limitrange -n <NAMESPACE> -o yaml
```

---

## 해결 가이드

- 리소스 부족 → Pod `requests`/`limits` 조정 또는 노드 증설
- PVC Pending → PVC 바인딩 상태 확인, 스토리지클래스 조건 점검
- 어피니티/테인트 → 조건 완화 또는 tolerations 추가
- 쿼터 초과 → 관리자에게 쿼터 상향 요청 또는 Pod 스펙 축소
- 이미지 풀 실패 → 레지스트리 인증/태그 확인, `imagePullSecrets` 점검
- 노드 상태 이상 → 노드 상태 복구(디스크/메모리 여유 확보, NotReady 원인 해결)
- 보안 정책 위반 → Pod 스펙을 정책에 맞게 수정하거나 정책 완화

---
