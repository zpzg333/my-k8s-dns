# my-k8s-dns

> Configmap 수정으로 간단하게 DNS 서버 설정. 

---

## 개요

베어메탈 Kubernetes 클러스터에서는 외부 DNS 서버가 없어 서비스 접근이 IP 직접 입력에 의존합니다.  
이 프로젝트는 **CoreDNS를 Kubernetes에 독립 배포**하여 홈랩 전용 내부 DNS를 구성합니다.  
서비스 추가·변경 시 ConfigMap 수정만으로 즉시 반영됩니다.

## 관리 도메인 목록

| 도메인 | 서비스 | IP |
|---|---|---|
| www.syargocd.com | ArgoCD (GitOps) | 192.168.203.70 |
| www.syansible.com | Ansible 자동화 | 192.168.203.63 |
| www.syk8smaster.com | K8s Master | 192.168.202.70 |
| www.syk8snode1.com | K8s Node1 | 192.168.202.72 |
| www.sydockerbuilder.com | Docker Build 서버 | 192.168.202.78 |
| www.sygrafana.com | Grafana | 192.168.203.64 |
| www.syprometheus.com | Prometheus | 192.168.203.67 |
| www.dwcts.cvp1.co.kr | CloudVision (CVP1) | 192.168.201.244 |
| www.dwcts.cvp2.co.kr | CloudVision (CVP2) | 192.168.201.245 |
| www.dwcts.cvp3.co.kr | CloudVision (CVP3) | 192.168.201.246 |

## 아키텍처

```
클라이언트 DNS 쿼리
        ↓
CoreDNS Pod (K8s)
        ↓
  hosts 플러그인 → 내부 도메인 응답
        ↓ (미매핑 도메인)
  kubernetes 플러그인 → K8s 서비스 DNS
        ↓ (외부 도메인)
  forward → 업스트림 DNS
```

## 기술 스택

| 항목 | 기술 |
|---|---|
| DNS 서버 | CoreDNS |
| 플랫폼 | Kubernetes |
| 설정 | ConfigMap (Corefile) |
| 노출 | LoadBalancer Service (MetalLB 연동) |

## 구조

```
my-k8s-dns/
├── DNS-Depolyment.yaml    ← CoreDNS Deployment
├── DNS-SVC.yaml           ← LoadBalancer Service
└── DNS-configmap.yaml     ← Corefile (도메인 매핑 설정)
```

## 실행 방법

```bash
kubectl create namespace dns-service

kubectl apply -f DNS-configmap.yaml
kubectl apply -f DNS-Depolyment.yaml
kubectl apply -f DNS-SVC.yaml

# DNS 동작 확인
kubectl get svc -n dns-service
nslookup www.syargocd.com <DNS_EXTERNAL_IP>
```

## 사용 기술 포인트

- **CoreDNS 플러그인 체인**: hosts → kubernetes → forward 순서로 DNS 쿼리 처리
- **Kubernetes 네이티브 운영**: CoreDNS를 K8s Deployment로 운영하여 HA 및 자동 복구 지원
- **MetalLB 연동**: LoadBalancer 타입 Service로 외부 접근 가능한 DNS IP 발급
- **CVP 도메인 통합**: Arista CloudVision 3노드 클러스터를 도메인으로 관리

## ArgoCD 연동

이 프로젝트는 **ArgoCD(GitOps)** 와 연동되어 있습니다.  
도메인 추가·변경 시 `DNS-configmap.yaml`만 수정하여 GitHub에 push하면  
ArgoCD가 자동으로 감지하여 CoreDNS ConfigMap을 클러스터에 반영합니다.

```
GitHub push (DNS-configmap.yaml 수정)
        ↓
ArgoCD 자동 감지 (auto-sync)
        ↓
CoreDNS ConfigMap 업데이트
        ↓
DNS 쿼리 즉시 반영 (재시작 없음)
```

| 항목 | 내용 |
|---|---|
| GitOps 도구 | ArgoCD |
| 배포 트리거 | GitHub push |
| 대상 | DNS-configmap.yaml, DNS-Depolyment.yaml, DNS-SVC.yaml |
| ArgoCD 서비스 | www.syargocd.com |

## 배경

홈랩에서 운영하는 Arista CVP, 모니터링 스택, CI/CD 툴을 IP 대신 도메인으로 일관되게 접근하기 위해 구성한 프로젝트입니다.
