# Ansible 명령어 사용하기
이 파트에서는 간단히 Ansible ad-hoc 명령어를 사용해 봅니다.

## Ansible ad-hoc을 이용해 간단히 명령어 실행하기
Ad-hoc Command 에 대하여 알아보기 <br>
https://docs.ansible.com/ansible/latest/user_guide/intro_adhoc.html

* Ansible이 강력한 이유는 Playbook을 사용하기 때문입니다.
* ad-hoc 명령어는 한번 명령어를 사용하고 버려도 되는 경우에 사용합니다.

```bash
# ansible HOST -m MODULE_NAME
$ ansible localhost -m ping
```
