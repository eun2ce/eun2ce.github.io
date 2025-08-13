---
title: "[ops] k3s 위에서 MIG 지원 GPU 모니터링 환경 구성하기"
description: "DCGM Exporter → Prometheus → Grafana(Rancher Monitoring)"
date: 2025-08-13 15:09:00 +0900
categories: [ "ops", "k8s" ]
tags: [ "k8s", "dcgm-exporter", "prometheus", "grafana", "gpu-driver-plugin" ]
pin: false
math: false
mermaid: false
image:
  path: /assets/img/posts/2025-08-13-gpu-monitoring-on-k3s-2025-08-13-10-38.webp
---

* 환경
    * k3s
    * nvidia-device-plugin
    * 단일노드 mig + gpu 섞어 사용

> gpu 사용량(시간별) + k3s 클러스터 모니터링

## TL;DR

1. **dcgm-exporter** DaemonSet 배포 (NVML 주입, Service 생성)  
2. **ServiceMonitor** 로 Prometheus 수집 연결  
3. **Prometheus & Grafana** 저장소(PV/PVC) 구성 + `retentionSize: "32GiB"`  
4. **Grafana Ingress** 생성 + `grafana.ini` 의 `server.root_url/domain` 을 **인그레스 호스트**로 고정  
5. 접속/대시보드 임포트(12239) → 시간별 GPU/메모리 확인  
6. (주의) Rancher 프록시 경로로 **리다이렉트되는 이슈**는 `server.root_url`/사이드카 설정으로 대부분 해결되지만, 일부 링크에서 잔여 리다이렉트가 보일 수 있음(문서 말미 트러블슈팅 참고)

## 사전 준비

```bash
# 노드에서 GPU/드라이버 동작
nvidia-smi

# k3s 기본 IngressClass (traefik)
kubectl get ingressclass

# rancher-monitoring 설치됨
kubectl -n cattle-monitoring-system get pods | grep -E 'prometheus|grafana|operator'
```
- **nvidia-device-plugin** 만 사용(= GPU Operator 미사용)
    - gpu operator는 mig와 gpu를 섞어 사용 할 경우 지원 x

## dcgm-exporter 배포 (deamonset + service)

- `runtimeClassName: nvidia`(권장) 또는 `nvidia.com/gpu` 리밋으로 **NVML 주입**
- `privileged: true`, `/var/lib/kubelet/pod-resources` 마운트
- 노드 라벨에 맞춘 `nodeSelector` (예: `gpu=nvidia`)

```yaml
# dcgm-exporter.yaml
```

## ServiceMonitor로 prometheus에 연결

> prometheus가 서비스 모니터를 전체 namespace에서 집계하도록 설정되어있어야 함

```yaml
# dcgm-servicemonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: dcgm-exporter
  namespace: kube-system
spec:
  namespaceSelector:
    matchNames: ["kube-system"]
  selector:
    matchLabels:
      app.kubernetes.io/name: "dcgm-exporter"
      app.kubernetes.io/version: "4.4.0"
  endpoints:
    - port: metrics
      interval: 30s
```

## prometheus & grafana pvc 구성

### PV/SC

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: prometheus-nas-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
allowVolumeExpansion: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: prometheus-pv-nas
  labels:
    pv: prometheus-nas
spec:
  capacity:
    storage: 32Gi
  accessModes: ["ReadWriteOnce"]
  storageClassName: prometheus-nas-sc
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /nas/prometheus
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values: ["axxx-4"]
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: grafana-nas-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Retain
allowVolumeExpansion: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: grafana-pv-nas
  labels:
    pv: grafana-nas
spec:
  capacity:
    storage: 10Gi
  accessModes: ["ReadWriteOnce"]
  storageClassName: grafana-nas-sc
  persistentVolumeReclaimPolicy: Retain
  local:
    path: /nas/grafana
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values: ["axxx-4"]
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: grafana-pvc
  namespace: cattle-monitoring-system
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: grafana-nas-sc
  resources:
    requests:
      storage: 10Gi
  selector:
    matchLabels:
      pv: grafana-nas

```

`rancher-monitoring` 값에서:
```yaml
prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: prometheus-nas-sc
          accessModes: ["ReadWriteOnce"]
          resources: { requests: { storage: 32Gi } }
          selector:
            matchLabels: { pv: prometheus-nas }
    retention: 10d
    retentionSize: "32GiB"
```

## grafana, prometheus 스토리지 연결

> 랜처를 사용하는 경우 app > installed app > monitoring 쪽에서 config 수정해도 무방

리소스가 많지 않기때문에 최대한 낮춰사용 > 필요하면 높게 할당해서 update

```yaml
grafana:
  persistence:
    enabled: true
    type: pvc
    existingClaim: grafana-pvc
  resources:
    requests:
      cpu: "50m"
      memory: "128Mi"
    limits:
      cpu: "300m"
      memory: "512Mi"

prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: prometheus-nas-sc
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 32Gi
          selector:
            matchLabels:
              pv: prometheus-nas
    resources:
      requests:
        cpu: "200m"
        memory: "512Mi"
      limits:
        cpu: "1"
        memory: "2Gi"
    walCompression: true
    retention: 15d
    retentionSize: "28Gi"

alertmanager:
  alertmanagerSpec:
    resources:
      requests:
        cpu: "50m"
        memory: "128Mi"
      limits:
        cpu: "300m"
        memory: "512Mi"
```

## (선택) ingress설정

랜처를 통해 모니터링을 구성하게 되면 아래와 비슷한 url로 접속하게 된다.  
`https://rancher.xxxx.nip.io/dashboard/c/local/apps/catalog.cattle.io.app/cattle-monitoring-system/rancher-monitoring#resources` 필자는 `grafana.xxxx.nip.io 형태로 접근하고싶어 아래와 같이 수정

rancher 기본값은 rancher ui 프록시 기준이며, ingress를 통해 직접 공개하려면 root ingress host로 고정해야 한다.

```ini
# grafana.ini (Values의 grafana.grafana.ini.server에 해당)
[server]
domain = grafana.xxxx.nip.io
root_url = http://grafana.xxxx.nip.io
serve_from_sub_path = false
```

grafana의 ingress 부분을 아래와 같이 수정

```yaml
grafana:
  ingress:
    enabled: true
    ingressClassName: traefik
    annotations:
      traefik.ingress.kubernetes.io/router.entrypoints: web
    hosts:
      - grafana.xxx.nip.io
    path: /
    pathType: Prefix
    tls: [] # 사용 할 경우 작성
```

## 결과

![](/assets/img/posts/2025-08-13-gpu-monitoring-on-k3s-2025-08-13-10-37.webp)
![](/assets/img/posts/2025-08-13-gpu-monitoring-on-k3s-2025-08-13-10-38.webp)