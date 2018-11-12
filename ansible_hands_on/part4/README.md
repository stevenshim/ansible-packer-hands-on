### Step 1 - ansible 기본 config
ansible 을 수행할 directory에 ansible.cfg 파일을 생성합니다.<br>
편의를 위해서 ec2-user 의 home directory에서 진행하시면 좋습니다.

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

서버에 접속할 인증키는 ~/.ssh/ansible-packer.pem 에 위치합니다.<br>

ansible을 수행할 directory에 my-inventory 파일을 생성합니다.<br>
```txt
[target]
YOUR_TARGET_EC2_IPs

# 예시
[target]
13.209.82.234
```

### Inventory에 대해서 좀 더 알아보기
만약 target이 하나가 아니라면 아래 처럼 여러개를 설정 혹은 그룹으로 지정할 수 있습니다.<br>
이 외에도 다양한 inventory 표현법이 있으니 공식 문서 {URL}을 확인 하시기 바랍니다.

```txt
[webserver]
10.100.0.1
10.100.0.10

[dbserver]
10.200.0.10
10.200.0.11
```

### Config 에 대해서 좀 더 알아보기
Ansible의 config는 config file 혹은 설정 있을 경우 아래 순서대로 적용됩니다. <br>
더 자세한 내용은 공식 문서 {URL}에서 확인 가능합니다.

* ansible 을 실행할 당시 runtime parameter
* ANSIBLE_CONFIG 이름의 환경변수
* Ansible 을 실행할 경로에 있는 ansible.cfg 파일
* ~/.ansible.cfg (Ansible 을 실행하는 user의 home directory)
* /etc/ansible/ansible.cfg
