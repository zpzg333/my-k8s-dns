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
