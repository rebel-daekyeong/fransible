# install
## Clone
git clone git@github.com:rebel-daekyeong/fransible.git --recurse-submodules

## Python 환경 설정
```
poetry sync
```

## Ansible 실행
```
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini -e ansible_user=<<user ssh id>>  playbooks/<your playbook.yaml -vv
```

# Test
소스 수정 후 -> playbooks/ 디렉토리 하위에 test playbook 생성 -> Ansible 실행
```
#test-playbooks.yaml
---
- name: Install Prometheus
  hosts: kube_control_plane[0]
  roles:
  - {role: test-role, tags: test-role}
```
```
ansible-playbook -i inventory/inventory.ini -i inventory/r18s.ini -e ansible_user=<user ssh id>  playbooks/test-playbooks.yaml -vv
```
