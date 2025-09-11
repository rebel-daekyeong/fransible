# Fransible - Framework Software Ansible

Rebellions Framework Software를 위한 Ansible 기반 인프라 관리 도구

## 📖 개요

Fransible은 Rebellions AI의 Framework Software 인프라를 자동화하고 관리하기 위한 Ansible 프로젝트입니다. Kubernetes 클러스터 배포, RBLN ATOM 가속기 드라이버 설치, 그리고 다양한 인프라 구성요소들의 설치 및 관리를 자동화합니다.

## 🏗️ 프로젝트 구조

```
fransible/
├── inventory/                    # 인벤토리 설정
│   ├── inventory.ini            # 메인 인벤토리 파일
│   ├── r18s.ini                # R18S 특화 인벤토리
│   ├── artifacts/              # Kubernetes 설정 파일들 (사용자별 kubeconfig, token)
│   ├── credentials/            # 민감한 인증 정보
│   └── group_vars/             # 그룹 변수 설정
├── playbooks/                   # 메인 플레이북들 (심볼릭 링크)
├── roles/                      # 커스텀 역할들
├── submodules/                 # Git 서브모듈들
│   ├── kubespray/              # Kubernetes 배포용 Kubespray
│   ├── fraybooks/              # 프레임워크 플레이북들
│   └── froles/                 # 프레임워크 역할들
├── docker/                     # Docker 컨테이너 설정
│   ├── Dockerfile              # 멀티스테이지 빌드 설정
│   ├── docker-compose.yaml     # 컨테이너 오케스트레이션
│   └── .env.example           # 환경 변수 템플릿
├── artifacts/                  # 빌드 아티팩트 및 출력물
├── pyproject.toml              # Python 의존성 관리 (Poetry)
├── poetry.lock                 # 의존성 락 파일
└── ansible.cfg                 # Ansible 설정
```

## 🚀 빠른 시작

### 1. 저장소 클론

```bash
git clone git@github.com:rebel-daekyeong/fransible.git --recurse-submodules
cd fransible
```

### 2. Python 환경 설정

이 프로젝트는 Poetry를 사용하여 의존성을 관리합니다.

```bash
# Poetry를 사용한 의존성 설치 (권장)
poetry install
poetry shell

# 또는 pip 사용 (Poetry가 없는 경우)
pip install -e .
```

### 3. Docker 환경 설정 (선택사항)

컨테이너화된 환경에서 Ansible을 실행할 수 있습니다:

```bash
# 환경 변수 설정
cp docker/.env.example docker/.env
# .env 파일을 편집하여 필요한 변수들을 설정

# Docker 이미지 빌드 및 실행
cd docker
docker compose build python
docker compose run python bash
```

### 4. 인벤토리 설정

`inventory/inventory.ini` 파일에서 관리할 호스트들을 설정합니다:

```ini
[all]
your-host-01    ansible_host=192.168.1.10
your-host-02    ansible_host=192.168.1.11

[atom]
your-host-01    accelerator=ca25
your-host-02    accelerator=ca22

[kube_control_plane]
your-host-01

[kube_node]
your-host-02
```

## 📚 주요 플레이북

### Kubernetes 클러스터 설정

```bash
# 전체 클러스터 설정
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-cluster.yaml -vv

# 클러스터 스케일링
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/scale-cluster.yaml -vv

# 클러스터 리셋
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/reset-cluster.yaml -vv
```

### RBLN ATOM 드라이버 설정

```bash
# ATOM 드라이버 설치
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-driver.yaml -vv
```

### Framework 구성요소 설정

```bash
# 패키지 설치 및 관리
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-packages.yaml -vv

# 사용자 관리
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-users.yaml -vv

# R18S 특화 설정
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-r18s.yaml -vv

# Karv 설정
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-karv.yaml -vv
```

## 🏷️ 호스트 그룹 분류

### 가속기 타입별 분류
- **atom**: RBLN ATOM 가속기가 설치된 호스트들
- **gpu**: NVIDIA GPU가 설치된 호스트들

### 환경별 분류
- **vm**: 가상머신 환경
- **r18s**: R18S 프로젝트 전용 호스트들
- **ci**: CI/CD 전용 호스트들
- **fsw**: Framework Software 관련 호스트들

### 네트워크 설정
- **rdma**: RDMA 네트워킹이 설정된 호스트들
- **proxy_jump**: Bastion 호스트를 통해 접근하는 호스트들

## 🔧 주요 역할(Roles)

### 인프라 구성요소
- **harbor**: Docker 레지스트리
- **prometheus**: 모니터링 시스템
- **cert-manager**: TLS 인증서 관리
- **metallb**: 로드밸런서
- **istio**: 서비스 메시

### 가속기 관련
- **atom**: RBLN ATOM 드라이버 설치 및 관리
- **rbln-operator**: Kubernetes RBLN 오퍼레이터
- **gpu-operator**: NVIDIA GPU 오퍼레이터

### 네트워킹
- **multus-cni**: 다중 네트워크 인터페이스
- **sriov-cni**: SR-IOV 네트워킹
- **rdma-cni**: RDMA 네트워킹

### 스토리지
- **nfs-server**: NFS 서버 설정
- **nfs-client**: NFS 클라이언트 설정
- **nfs-subdir-external-provisioner**: NFS 동적 프로비저닝
- **local-pv**: 로컬 퍼시스턴트 볼륨

### 개발 도구
- **gitea**: Git 저장소 서버
- **nexus3**: 아티팩트 저장소
- **actions-runner-controller**: GitHub Actions 러너

### ML/AI 워크로드
- **kuberay**: Ray 클러스터 오퍼레이터
- **sample-app**: 샘플 애플리케이션 배포

### 보안 및 정책
- **kyverno**: Kubernetes 정책 엔진

## 🧪 개발 및 테스트

### 테스트 플레이북 생성

소스 수정 후 테스트를 위한 플레이북을 생성합니다:

```yaml
# playbooks/test-playbook.yaml
---
- name: Test Custom Role
  hosts: test-group
  vars:
    test_variable: "test_value"
  roles:
    - {role: test-role, tags: test-role}
```

### 테스트 실행

```bash
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/test-playbook.yaml -vv
```

### 특정 태그만 실행

```bash
# 특정 태그만 실행
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-cluster.yaml \
  --tags="docker,kubelet" -vv

# 특정 태그 제외
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini \
  -e ansible_user=<ssh_user> playbooks/setup-cluster.yaml \
  --skip-tags="etcd" -vv
```

## 📋 요구사항

### 시스템 요구사항
- Python 3.10 이상 3.13 미만
- Ansible 10.x
- SSH 키 기반 인증 설정

### Python 의존성
주요 의존성 패키지들:
- **ansible** (>=10,<11): Ansible 코어
- **ansible-dev-tools** (>=25.8.3,<26.0.0): 개발 도구
- **ansible-lint** (>=25.8.1,<26.0.0): 코드 품질 검사
- **kubernetes** (>=33.1.0,<34.0.0): Kubernetes 클라이언트
- **netaddr** (>=1.3.0,<2.0.0): 네트워크 주소 조작
- **jmespath** (>=1.0.1,<2.0.0): JSON 쿼리 언어
- **toml** (>=0.10.2,<0.11.0): TOML 파일 파싱
- **poetry** (==2.0.1): 의존성 관리 도구

## 🐳 Docker 지원

이 프로젝트는 멀티스테이지 Docker 빌드를 지원합니다:

### 사용 가능한 이미지 타겟

- **driver**: RBLN ATOM 드라이버가 포함된 베이스 이미지
- **python**: Python 환경과 기본 패키지들
- **build**: 빌드 도구와 컴파일 환경
- **full**: 모든 구성요소가 포함된 완전한 이미지 (주석 처리됨)

### Docker 사용법

```bash
# Python 환경으로 실행
docker compose run python bash

# 빌드 환경으로 실행
docker compose run build bash

# 드라이버 환경으로 실행
docker compose run driver bash
```

### 환경 변수 설정

`docker/.env` 파일에서 다음 변수들을 설정할 수 있습니다:
- `REPO`: Docker 레지스트리 이름
- `TAG`: 이미지 태그
- `PLATFORM`: 플랫폼 아키텍처
- `ATOM_DRIVER_VERSION`: ATOM 드라이버 버전
- `RBLN_CCL_VERSION`: RBLN CCL 버전
- 기타 버전 관련 변수들

## 👥 사용자 관리

프로젝트는 다음 사용자들의 Kubernetes 설정을 관리합니다:

### 관리되는 사용자 목록
- chanheo, daekyeong, hongseok, jaebin, jaehwang-jung
- jindol21, jinmoo, jiwoo, jongwonlee, minwook
- rebel-clee, scottlee, seungchul, wonsubkim, ykchoi
- ci-bot (CI/CD 전용 봇 계정)

### Kubernetes 설정 파일들
각 사용자별로 다음 파일들이 `inventory/artifacts/`에 저장됩니다:
- `{username}.config`: Kubernetes 설정 파일
- `{username}.token`: 인증 토큰
- `admin.conf`: 클러스터 관리자 설정

## 🔍 문제 해결

### 일반적인 문제들

1. **SSH 연결 실패**
   - SSH 키가 올바르게 설정되었는지 확인
   - `ansible_user` 변수가 올바른지 확인
   - Bastion 호스트를 통한 접근이 필요한 경우 `bastion_host` 설정 확인

2. **권한 부족 오류**
   - `become: true` 설정 확인
   - sudo 권한이 있는지 확인

3. **ATOM 드라이버 설치 실패**
   - 드라이버 파일 경로 확인
   - 펌웨어 업데이트가 필요한지 확인
   - Docker 환경에서는 적절한 이미지 타겟 사용

4. **Docker 관련 문제**
   - `.env` 파일이 올바르게 설정되었는지 확인
   - Docker secrets가 설정되었는지 확인
   - 필요한 패키지 디렉토리 경로가 존재하는지 확인

5. **사용자 설정 문제**
   - `inventory/artifacts/`에 필요한 사용자 설정 파일들이 있는지 확인
   - Kubernetes 클러스터에 접근 권한이 있는지 확인

### 로그 확인

```bash
# 상세한 로그 출력
ansible-playbook -vvv ...

# 특정 호스트만 대상으로 실행
ansible-playbook --limit=host-name ...

# Docker 컨테이너에서 실행
docker compose run python ansible-playbook -i inventory/inventory.ini ...
```

## 🤝 기여하기

1. 이슈 리포트나 기능 요청을 위해 GitHub Issues를 사용해주세요
2. Pull Request는 언제나 환영입니다
3. 코드 변경 시 관련 테스트를 함께 제출해주세요

## 📄 라이선스

이 프로젝트는 Rebellions AI의 내부 프로젝트입니다.

## 👥 메인테이너

- **Daekyeong Kim** - daekyeong.kim@rebellions.ai
