
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
### Template 작성
packer를 수행할 폴더에서 template file을 생성한다.

##### packer_hands_on/basic-template.json
```json
{
  "builders": [{
    "type": "amazon-ebs",
    "subnet_id": "{{user `aws_subnet_id`}}",
    "vpc_id": "{{user `aws_vpc_id`}}",
    "region": "{{user `aws_region`}}",
    "security_group_id": "{{user `security_group_id`}}",
    "ssh_keypair_name": "ansible-packer",
    "ssh_username": "ec2-user",
    "ssh_private_key_file": "ansible-packer.pem",
    "ssh_pty": "false",
    "ssh_interface": "private_ip",
    "instance_type": "t2.micro",
    "source_ami": "{{user `aws_source_ami`}}",
    "ami_name": "{{user `aws_target_ami`}}",
    "associate_public_ip_address": "true",
    "tags": {"service": "{{user `service`}}"},
    "run_tags": {
        "Name": "{{user `ec2-name`}}",
        "Cost-Center": "awskrug"
    },
    "ami_block_device_mappings": [
      {
        "device_name": "/dev/sda1",
        "delete_on_termination": "true",
        "volume_type": "gp2"
      }
    ]
  }],
 "provisioners": [{
   "type": "ansible",
   "playbook_file": "/home/ec2-user/playbook/httpd.yml",
   "user": "ec2-user",
   "sftp_command": "/usr/libexec/openssh/sftp-server",
   "extra_arguments": [
     "--extra-vars",
     "service={{user `service`}}"
   ]
 }]
}
```
### variable file 작성
template에 변수 처리 할 내용을 variable로 작성할 수 있다.

##### packer_hands_on/var-file.json
```json
{
  "aws_vpc_id": "vpc-c10536a9",
  "aws_subnet_id": "subnet-65a6910d",
  "security_group_id": "sg-0b26500a049ce59cb",
  "aws_source_ami": "ami-0a10b2721688ce9d2",
  "aws_target_ami": "my-first-ami",
  "service": "awskrug",
  "ec2-name": "now-baking"
}
```









<br><br><br><br><br><br><br><br><br><br><br><br>
