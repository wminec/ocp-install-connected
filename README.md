# OpenShift Agent-Based Bastion Setup (Ansible)

> **Environment**
>
> - Rocky Linux **9.5**
> - Python **3.9**
> - Ansible **core 2.14**

Bastion에서 **로컬 실행**하여 OpenShift를 **Agent-Based** 방식으로 설치하기 위한 준비 작업(Pre/Post 포함)을 자동화합니다.

---

## What this playbook does

- 필수 패키지 설치: `openssl`, `haproxy`, `bind`(또는 `bind9`), `wget`, `nmstate`, `bash-completion`
- SELinux/Firewalld 비활성화(옵션)
- **HAProxy** 구성 (SNO/HA 템플릿 분기)
- **BIND (named)** 구성: `named.conf`, `named.rfc1912.zones`, **forward zone** 템플릿 배포 + 백업/검증
- **AgentConfig** (`agent-config.yaml`) 생성 (+ `install-dir`로 복사)
- **InstallConfig** (`install-config.yaml`) 생성  
  - `single_node: true` → `controlPlane.replicas=1`, `compute.replicas=0`  
  - `single_node: false` → `controlPlane.replicas=3`, `compute.replicas=workers 개수`
  - SSH 키 자동 생성/활용, Pull-secret(파일 또는 변수) 지원
- **Chrony MachineConfig**: Butane `.bu` → YAML 변환 → `install-dir/openshift/` 배치
- **Agent ISO** 생성: `openshift-install agent create image`
- Post-install 편의:
  - `oc` bash-completion 설치
  - `~/.kube/config` 구성
  - **DNS는 실제 변경하지 않고** 안내 메시지 출력  
    `Change the bastion's DNS server to the bastion itself (nameserver <bastion_ip>).`
  - `kubeadmin-password` 출력

---

# 모든 단계 실행
```bash
ansible-playbook site.yml
```

# Notes
네트워크/DNS/보안 정책은 환경에 맞게 조정하세요.