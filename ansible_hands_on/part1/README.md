# EC2 접속 및 Ansible 설치하기
Part1 에서는 EC2 접속 및 Ansible 설치를 진행합니다.

## EC2 접속하기
Prerequisites의 2번째 과정에서 생성한 EC2에 접속을 시도합니다.

##### Mac / Linux
```bash
## {중괄호 부분은 여러분이 생성한 Key pair, EC2 IP에 맞게 변경하세요}
$ ssh -i {YOUR_KEY_PAIR.pem} ec2-user@{YOUR_EC_PUBLIC_IP}

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

## Ansible 설치 하기
Ansible 공식 설치 메뉴얼은 https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html 에서 확인 가능하며, apt-get, yum, pip, brew 등 다양한 방법으로 설치가 가능합니다.

##### Python pip을 이용한 Ansible 설치하기
```bash
## sudo 권한 필요
$ sudo pip install ansible

## 설치 확인
$ ansible --version

$ which ansible
# /usr/local/bin/ansible
```
