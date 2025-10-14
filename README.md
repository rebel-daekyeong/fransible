# Fransible - Framework Software Ansible

AI 인프라를 위한 Ansible 기반 자동화 관리 도구

## 개요

Fransible은 AI 워크로드를 위한 인프라를 자동화하고 관리하기 위한
Ansible 프로젝트입니다. Kubernetes 클러스터 배포(Kubespray 활용),
AI 가속기 드라이버 설치, 그리고 다양한 인프라 구성요소들의 설치 및
관리를 자동화합니다.

## 빠른 시작

### 사전 요구사항

- Python 3.10+ (< 3.13)
- Ansible 10.x
- SSH 키 기반 인증
- Git (서브모듈 지원)

### 설치

```bash
# 1. 저장소 클론
git clone <repository-url> --recurse-submodules
cd fransible

# 2. Python 환경 설정 (Poetry 권장)
poetry install
poetry shell

# 3. 인벤토리 설정
cp -r inventory/sample/ my_inventory/
# my_inventory/inventory.ini 파일 편집
```

### 기본 사용법

```bash
# Kubernetes 클러스터 배포
ansible-playbook \
  -i inventory/inventory.ini \
  -i inventory/my_inventory/inventory.ini \
  -i inventory/my_inventory/kubespray.ini \
  playbooks/setup-cluster.yaml

# 드라이버 설치
ansible-playbook \
  -i inventory/inventory.ini \
  -i inventory/my_inventory/inventory.ini \
  playbooks/setup-driver.yaml

# 패키지 및 사용자 설정
ansible-playbook \
  -i inventory/inventory.ini \
  -i inventory/my_inventory/inventory.ini \
  playbooks/setup-packages.yaml
ansible-playbook  \
  -i inventory/inventory.ini \
  -i inventory/my_inventory/inventory.ini \
  playbooks/setup-users.yaml
```

### 인벤토리 구성 및 사용법

이 프로젝트는 조직/환경별로 계층화된 인벤토리 구조를 사용합니다.
각 환경은 독립적인 인벤토리 파일과 그룹 변수를 가지고 있으며,
**다중 인벤토리 옵션(`-i`)을 사용하여 설정을 오버라이드할 수 있습니다.**

#### 인벤토리 구조

```text
inventory/
├── inventory.ini              # 기본 인벤토리
├── sample/                    # 샘플 환경
│   ├── inventory.ini
│   ├── kubespray.ini          # Kubespray용 인벤토리
│   └── group_vars/
├── fsw/                       # FSW 조직
│   ├── inventory.ini
│   ├── r18s/                  # R18S 프로젝트
│   │   ├── inventory.ini
│   │   ├── kubespray.ini
│   │   ├── clusterware/       # Clusterware 환경
│   │   │   ├── inventory.ini
│   │   │   └── kubespray.ini
│   │   ├── dev/               # 개발 환경
│   │   │   ├── inventory.ini
│   │   │   └── kubespray.ini
│   │   └── dk_test/           # 테스트 환경
│   │       ├── inventory.ini
│   │       └── kubespray.ini
│   └── group_vars/
└── [기타 조직별 디렉토리]
```

#### 인벤토리 파일 설명

- **inventory.ini**: 일반 Ansible 플레이북용 호스트 및 그룹 정의
- **kubespray.ini**: Kubespray 전용 인벤토리 (클러스터 배포 시 필수)

#### 인벤토리 우선순위

Ansible은 나중에 지정된 인벤토리가 이전 인벤토리의 설정을 오버라이드합니다:

1. 첫 번째 `-i`: 기본 설정
1. 중간 `-i`: 조직/프로젝트/환경별 설정으로 순차 오버라이드
1. 마지막 `-i`: Kubespray 인벤토리 (클러스터 배포 시)

## 주요 구성요소

### playbooks

- `setup-cluster.yaml`: Kubernetes 클러스터 설정 (kubespray.ini 필수)
- `scale-cluster.yaml`: 클러스터 노드 추가/제거 (kubespray.ini 필수)
- `reset-cluster.yaml`: 클러스터 초기화 (kubespray.ini 필수)
- `setup-driver.yaml`: AI 가속기 드라이버 설치
- `setup-packages.yaml`: 시스템 패키지 관리
- `setup-users.yaml`: 사용자 계정 관리

### roles

#### 클러스터 및 컨테이너

- **Kubernetes 관리**: 클러스터 배포, 네임스페이스, 매니페스트 관리
- **컨테이너 레지스트리**: Harbor, Gitea, Nexus3
- **네트워킹**: Multus-CNI, SR-IOV, RDMA, MetalLB
- **스토리지**: NFS, Local PV, 동적 프로비저닝

#### AI/ML 워크로드

- **가속기 지원**: GPU/NPU 드라이버 및 오퍼레이터
- **ML 프레임워크**: KubeRay, PyTorch 런타임
- **워크로드 관리**: 리소스 스케줄링 및 최적화

#### 모니터링 및 보안

- **모니터링**: Prometheus, Grafana, Loki
- **서비스 메시**: Istio, 도메인 라우팅
- **보안**: Kyverno, Cert-Manager, Secret 관리

#### CI/CD

- **GitHub Actions**: 러너 컨트롤러 및 스케일 세트
- **아티팩트 관리**: 이미지 레지스트리, 패키지 저장소

## 디렉토리 구조

```text
fransible/
├── inventory/          # 환경별 인벤토리 설정
│   ├── <org>/          # 조직별 디렉토리
│   │   └── <project>/  # 프로젝트별 디렉토리
│   │       └── <env>/  # 환경별 디렉토리
│   │           ├── inventory.ini
│   │           └── kubespray.ini
├── playbooks/          # 메인 플레이북
├── roles/              # Ansible 역할
├── submodules/         # Git 서브모듈 (Kubespray 등)
├── docker/             # 컨테이너 개발 환경
├── pyproject.toml      # Python 의존성 (Poetry)
└── ansible.cfg         # Ansible 설정
```

## Docker 개발 환경

```bash
cd docker/devpack
cp .env.example .env
docker compose build
docker compose run devpack bash
```

## 🧪 테스트

```bash
# Ansible 문법 검사
ansible-playbook playbooks/setup-cluster.yaml --syntax-check

# Lint 실행
ansible-lint playbooks/

# 역할 테스트
ansible-playbook playbooks/tests/test-prometheus-role.yaml

# 인벤토리 확인
ansible-inventory --list \
  -i inventory/fsw/r18s/dev/inventory.ini \
  -i inventory/fsw/r18s/dev/kubespray.ini
```

## 문제 해결

### 일반적인 이슈

1. **SSH 연결 문제**
   - SSH 키 권한 확인: `chmod 600 ~/.ssh/id_rsa`
   - 인벤토리의 `ansible_user` 설정 확인

1. **Kubernetes API 접근 오류**
   - kubeconfig 경로 확인
   - RBAC 권한 확인

1. **의존성 오류**
   - Python 버전 확인: `python --version`
   - 의존성 재설치: `poetry install --sync`

1. **인벤토리 오버라이드 문제**
   - 인벤토리 파일 순서 확인 (나중 파일이 우선)
   - `ansible-inventory --list` 명령으로 최종 병합 결과 확인
   - kubespray.ini 포함 여부 확인

### 자세한 디버깅

```bash
# 상세 로그
ansible-playbook -vvv ...

# 디버그 모드
ANSIBLE_DEBUG=1 ansible-playbook ...

# 인벤토리 병합 결과 확인
ansible-inventory --list \
  -i inv1.ini -i inv2.ini -i kubespray.ini

# 호스트 연결 테스트
ansible -i inventory/<path>/inventory.ini all -m ping
```

## 문서

- [Ansible 공식 문서](https://docs.ansible.com/)
- [Kubespray 문서](https://kubespray.io/)
- [Poetry 문서](https://python-poetry.org/)
- [Ansible 인벤토리 문서](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html)

---

**Version**: 0.1.0
**Last Updated**: 2025-10
