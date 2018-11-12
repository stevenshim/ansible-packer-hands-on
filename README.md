# 준비하기
### 네트워크 준비하기
Cloud Formation Template을 이용해 네트워크(VPC) 준비하기 <br>
VPC 구성은 Public Subnet 2개 / Private Subnet 2개 / NAT Gateway 1개(비용 발생) 로 구성됩니다. <br>
테스트할 VPC 가 있는 분은 따로 진행하지 않으셔도 됩니다.

### EC2 생성하기
초기에 EC2는 2대 생성 합니다.<br>
1대는 ansible playbook을 작성하고 관리하는 ansible-server. <br>
다른 1대는 ansible playbook을 이용해 provisioning 할 목적의 target-server.

1. AWS Console 접속
2. 서울 리전(ap-northeast-2)으로 이동
3. AWS EC2 Console 로 이동
4. EC2 생성하기 - Amazon Linux AMI 2018.03.0 으로 생성
5. EC2 Type - t2.micro 로 선택
6. 기 생성된 VPC 중 Public Subnet 에 위치하도록 선택
7. Auto-assign Public IP 옵션 True
8. Security Group 에 22번 포트 접속 가능하도록 설정
9. Keypair 생성 후 다운로드 - 편의를 위해 2대 모두 같은 keypair로 선택.
10. EC2 생성 완료

### EC2 접속하기
##### Mac / Linux
```bash
## {중괄호 부분은 여러분이 생성한 Key pair, EC2 IP에 맞게 변경하세요}
$ ssh -i {YOUR_KEY_PAIR.pem} ec2-user@{10.100.0.0}

# 예시
$ ssh -i aws-krug-handson.pem ec2-user@10.100.10.1
```

##### Windows
1. Putty 다운로드<br>
https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
2. PuTTYgen을 이용하여 pem 키를 ppk (putty private key)로 변경하기<br>
https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/putty.html#putty-private-key
3. Putty 를 이용해 ec2 에 접속하기 <br>
https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/putty.html#putty-ssh

# Ansible 설치 및 사용하기
Ansible 공식 설치 메뉴얼은 https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html 에서 확인 가능<br>
apt-get, yum, pip, brew 등 다양한 방법으로 설치가 가능함.

##### Python pip을 이용한 Ansible 설치하기
```bash
## sudo 권한 필요
$ sudo pip install ansible

## 설치 확인
$ ansible --version

$ which ansible
# /usr/local/bin/ansible
```

### Ansible 을 이용해 명령어 실행 해보기
Ad-hoc Command 에 대하여 알아보기 <br>
https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html

* Ansible이 강력한 이유는 Playbook을 사용하기 때문
* ad-hoc 명령어는 한번 명령어를 사용하고 버려도 되는 경우에 사용함 (거의 쓸일이 없기는 함)

```bash
# ansible HOST -m MODULE_NAME
$ ansible localhost -m ping
```

### Playbook 구조 만들기
Playbook 디렉토리를 생성하고 각 Role별 Tasks, Templates, files 등 디렉토리 구조를 만든다.
```bash
## Ansible Playbook 을 관리할 디렉토리로 이동.
$ cd {your directory}

$ mkdir -p playbook/roles/httpd/
$ cd playbook/roles/httpd/
$ mkdir files handlers defaults templates tasks
$ cd -
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

### Step 1 - ansible 기본 config
ansible 을 수행할 directory에 ansible.cfg 파일을 생성한다.
```txt
[defaults]
host_key_checking = False
become_user = root
remote_user = ec2-user
timeout = 60000
callback_whitelist = profile_tasks
private_key_file = ~/.ssh/ansible-packer.pem
inventory = ./my-inventory

[ssh_connection]
pipelining = True
```

서버에 접속할 인증키를 ~/.ssh/ansible-packer.pem 에 위치한다.

ansible을 수행할 directory에 my-inventory 파일을 생성한다.
```txt
[target]
13.209.82.234
```

### Step 2 - Apache 기본 설치 / 루프 사용하기
```bash
$ vim playbook/roles/httpd/tasks/main.yml
```
```yaml
---
- name: install httpd with yum
  yum: name={{ item.package }}
  loop:
  - { package: httpd }
  - { package: mod_ssl }
```
```bash
# ansible-playbook --connection=local -i "127.0.0.1," playbook/httpd.yml
$ ansible-playbook -l target playbook/httpd.yml
```

### Step 3 - 변수 적용
##### playbook/roles/httpd/defaults/main.yml 에 변수 선언
```yaml
---
service: awskrug
```
##### playbook/roles/httpd/tasks/main.yml 에 스크립트 추가
```yaml
### 기존 작성 생략

### {{ 변수명 }} 으로 변수 처리
- name: create apache document root directory
  file:
    path: "/data/apache/{{ service }}/docroot"
    state: directory
    owner: apache
    group: apache
    mode: 0755
    recurse: yes
```

```bash
# Ansible Playbook 실행해보기
$ ansible-playbook -l target playbook/httpd.yml
```

### Step 4 - Templates 적용하기
##### playbook/roles/httpd/defaults/main.yml 에 변수 추가
```yaml
---
service: awskrug
http_port: 10180
```

##### playbook/roles/httpd/templates/http-awskrug-vhost.conf.j2 에 변수 추가
```txt
<VirtualHost *:{{ http_port }}>

  DocumentRoot /data/apache/{{ service }}/docroot

  ErrorLog     /log/apache/{{ service }}/{{ service }}.error_log
  CustomLog    /log/apache/{{ service }}/{{ service }}.access_log common

</VirtualHost>
```

##### playbook/roles/httpd/tasks/main.yml 에 templates / lineinfile 모듈 명령어 추가
```yaml
### 기존 작성 생략

### template을 이용해 변수 적용
- name: copy apache virtual host configuration
  template:
    src: "http-{{ service }}-vhost.conf.j2"
    dest: "/etc/httpd/conf.d/http-{{ service }}-vhost.conf"
    owner: apache
    group: apache
    mode: 0644

### lineinfile 을 이용해 파일 내 특정 라인만 내용 변경
- name: change apache default listening port configuration
  lineinfile:
    dest: /etc/httpd/conf/httpd.conf
    regexp: "^Listen"
    line: "Listen {{ http_port }}"
```

```bash
# Ansible Playbook 실행해보기
$ ansible-playbook -l target playbook/httpd.yml
```

##### cli 명령 파라미터로 변수 override 해보기
```bash
# Ansible Playbook 실행해보기
### -e http_port=20080 옵션 추가
$ ansible-playbook -l target playbook/httpd.yml -e http_port=20080
```

### Step 5 - Handlers 를 이용해 apache 재시작 하기
##### playbook/roles/httpd/handlers/main.yml 에 handler script 작성
```yaml
---
- name: restart apache webserver
  service:
    name: httpd
    state: restarted
```
##### playbook/roles/httpd/tasks/main.yml 에 notify 모듈 명령어 추가
```yaml
### 기존 아래 이름의 Task에 notify 추가
- name: copy apache virtual host configuration
  template:
    src: "http-{{ service }}-vhost.conf.j2"
    dest: /etc/httpd/conf.d/http-vhost.conf
    owner: apache
    group: apache
    mode: 0644
  notify:
    - restart apache webserver
```

```bash
# Ansible Playbook 실행해보기
$ ansible-playbook -l target playbook/httpd.yml

PLAY [all] ***************************************************************************

TASK [Gathering Facts] ***************************************************************
ok: [127.0.0.1]

TASK [httpd : install httpd] *********************************************************
ok: [127.0.0.1] => (item={u'package': u'httpd'})
ok: [127.0.0.1] => (item={u'package': u'mod_ssl'})

TASK [httpd : create apache document root directory] *********************************
ok: [127.0.0.1]

## TASK에 변경(changed)이 있으면,..
TASK [httpd : copy apache virtual host configuration] ********************************
changed: [127.0.0.1]

## HANDLER가 수행됨.
RUNNING HANDLER [httpd : restart apache webserver] ***********************************
changed: [127.0.0.1]

PLAY RECAP ***************************************************************************
127.0.0.1                  : ok=5    changed=1    unreachable=0    failed=0
```

### Step 6 - Handlers에 대해 좀 더 이해하기
notify 코드 이후에 더 task 를 추가해 봅니다.

##### playbook/roles/httpd/tasks/main.yml 에 notify 모듈 명령어 추가
```yaml
## 기존 코드 생략

- name: create apache log directory
  file:
    path: "/log/apache/{{ service }}/"
    state: directory
    owner: apache
    group: apache
    mode: 0755
    recurse: yes
```

```bash
# Ansible Playbook 실행해보기
$ ansible-playbook -l target playbook/httpd.yml

PLAY [all] ***************************************************************************

TASK [Gathering Facts] ***************************************************************
ok: [127.0.0.1]

TASK [httpd : install httpd] *********************************************************
ok: [127.0.0.1] => (item={u'package': u'httpd'})
ok: [127.0.0.1] => (item={u'package': u'mod_ssl'})

TASK [httpd : create apache document root directory] *********************************
ok: [127.0.0.1]

TASK [httpd : copy apache virtual host configuration] ********************************
changed: [127.0.0.1]

TASK [httpd : change apache default listening port configuration] ********************
ok: [127.0.0.1]

TASK [httpd : create apache log directory] *******************************************
ok: [127.0.0.1]

## Handler가 가장 마지막에 실행 됨
RUNNING HANDLER [httpd : restart apache webserver] ***********************************
changed: [127.0.0.1]

PLAY RECAP ***************************************************************************
127.0.0.1                  : ok=7    changed=2    unreachable=0    failed=0

```
notify에 등록된 handler는 바로 실행되지 않고, tasks 실행이 끝난 뒤 마지막에 수행 됨.<br>
원하는 지점에서 실행하게 하려면 아래처럼 코드를 추가 해야 함.
```yaml
### 기존 코드 notify 아래 - meta: flush_handlers 추가
- name: copy apache virtual host configuration
  template:
    src: "http-{{ service }}-vhost.conf.j2"
    dest: /etc/httpd/conf.d/http-vhost.conf
    owner: apache
    group: apache
    mode: 0644
  notify:
    - restart apache webserver

- meta: flush_handlers
```
```bash
# Ansible Playbook 실행해보기
$ ansible-playbook -l target playbook/httpd.yml

PLAY [all] ***************************************************************************

TASK [Gathering Facts] ***************************************************************
ok: [127.0.0.1]

TASK [httpd : install httpd] *********************************************************
ok: [127.0.0.1] => (item={u'package': u'httpd'})
ok: [127.0.0.1] => (item={u'package': u'mod_ssl'})

TASK [httpd : create apache document root directory] *********************************
ok: [127.0.0.1]

TASK [httpd : copy apache virtual host configuration] ********************************
changed: [127.0.0.1]

## flush 시점에 쌓여있던 handler trigger가 모두 수행됨.
RUNNING HANDLER [httpd : restart apache webserver] ***********************************
changed: [127.0.0.1]

TASK [httpd : change apache default listening port configuration] ********************
ok: [127.0.0.1]

TASK [httpd : create apache log directory] *******************************************
ok: [127.0.0.1]

PLAY RECAP ***************************************************************************
127.0.0.1                  : ok=7    changed=2    unreachable=0    failed=0

```

# Packer 설치하기
Ansible 코드를 작성하여 packer를 설치해 봅니다.

role 추가하기
```bash
$ mkdir -p playbook/roles/packer/tasks/
```
###### playbook/roles/packer/tasks/main.yml
```yaml
---
- name: download packer binary
  get_url:
    url: https://releases.hashicorp.com/packer/1.3.2/packer_1.3.2_linux_amd64.zip
    dest: /home/ec2-user/packer.zip
    mode: 0644

- name: unzip packer binary
  unarchive:
    src: /home/ec2-user/packer.zip
    dest: /usr/bin/
    remote_src: yes
    mode: 0755
  become: yes
  become_user: root
```
###### playbook/packer.yml
```yaml
---
- hosts: all
  roles:
  - packer
```

###### 실행하기
```bash
$ ansible-playbook -l target playbook/packer.yml
```

# Packer template 작성하기







<br><br><br><br><br><br><br><br><br><br><br><br>
