# Playbook 구조 만들기
Playbook 디렉토리를 생성하고 각 Role별 Tasks, Templates, files 등 디렉토리 구조를 만든다.
```bash
## Ansible Playbook 을 관리할 디렉토리로 이동.
## 실습 예제에서는 이해를 돕기 위해 /home/ec2-user 위치에서 아래 작업을 합니다.
$ cd {your directory}

$ mkdir -p playbook/roles/httpd/
$ cd playbook/roles/httpd/

# 이해를 돕기 위해 현재 directory 위치를 확인해 봅니다.
$ pwd
#/home/ec-2user/playbook/roles/httpd/

# 추가 directory를 생성 해봅니다.
# 각 directory는 ansible module별로 알맞은 역할을 하게 됩니다.
$ mkdir files handlers defaults templates tasks

# 디렉토리 생성이 끝났으니 다시 최초  
$ cd /home/ec2-user
$ tree
.
└── playbook
    └── roles
        └── httpd
            ├── defaults
            ├── files
            ├── handlers
            ├── tasks
            └── templates
```
