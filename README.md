# my-k8s-dns

# 컨셉
> Configmap 수정 -> K8S config map > 변경 으로 간단하게 DNS 서버 설정 관리. 

---

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
