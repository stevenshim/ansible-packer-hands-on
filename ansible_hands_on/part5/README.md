# Apache 기본 설치 / 루프 사용하기
이번 예제에서는 apache 서버를 설치하는 스크립트를 작성헤보도록 하겠습니다.<br>
여러개의 package를 한 task 를 이용해 설치하고 싶다면 `loop` 명령을 사용하여 손쉽게 해결할 수 있습니다.<br>
이번 예제에서는 apache 기본 설치와 함께 `loop` 명령어를 활용해보도록 합니다.

아래와 같이 main.yml을 tasks 디렉토리 아래 생성합니다.
```bash
$ vim playbook/roles/httpd/tasks/main.yml
```

### Apache Webserver를 설치하는 script 작성하기
vim 에디터에서 아래와 같이 ansible install script를 작성합니다.
```yaml
---
- name: install httpd with yum
  yum: name={{ item.package }}
  loop:
  - { package: httpd }
  - { package: mod_ssl }
```
위에서 `yum` 예약어는 yum package 관리자를 사용하기 위한 예약어입니다. <br>
만약 여러분들이 Ansible 을 활용해 provisioning 할 OS가 `apt`를 사용한다면 `apt` 예약어를 사용하면 됩니다. <br>
이번 hands-on에서는 다루지 않지만, 한번 작성하여 OS마다 다르게 사용하시려는 경우는 `when` 을 이용하여 분기가 가능합니다.<br>
좀 더 자세한 내용을 알고 싶으시다면 {URL}을 확인해주세요.

### Apache Webserver 관련 Role 작성하기
playbook/roles/httpd/ 디렉토리 및 그 아래 스크립트는 Apache webserver를 설치하는 하나의 role입니다.<br>
이 단계에서는 실행할 roles를 명시한 script를 작성합니다.

vim을 이용해 playbook 아래 httpd.yml 파일을 생성합니다.
```bash
$ vim ~/playbook/httpd.yam
```
```bash
---
- hosts: all
  become: yes
  become_user: root
  roles:
  - httpd
```

playbook의 장점은 여러 role을 조합하여 재사용 할 수 있는 점입니다. <br>
이 예제에서는 하나의 role을 만들겠지만, 실제 production 환경에서는 보다 복잡한 사용이 가능합니다.<br>
예를 들어 apache webserver가 설치된 target에 동시에


### 작성한 Ansible script 를 실행하기
Vim 에디터를 닫고 command line 명령어로 아래와 같이 실행을 합니다.<br>
편의를 위해서 이 명령어를 실행하는 위치 및 playbook 위치는 아래로 가정합니다.

아래 명령어의 -l 옵션은 inventory의 group 명을 지정하는 옵션입니다.<br>
우리는 이전에 my-inventory 파일을 생성할 때, [target] 이란 그룹에 target ec2의 ip를 넣어놨었습니다.

```bash
# user home directory로 이동
$ cd ~/

$ ansible-playbook -l target playbook/httpd.yml
```

이렇게 첫 playbook script를 실행해보았습니다. <br>
아래 내용은 실행 결과입니다.

```txt
PLAY [all] *******************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************
Thursday 20 December 2018  08:47:29 +0000 (0:00:00.070)       0:00:00.070 *****
ok: [10.100.0.159]

TASK [httpd : install httpd with yum] ****************************************************************************
Thursday 20 December 2018  08:47:29 +0000 (0:00:00.783)       0:00:00.854 *****
changed: [10.100.0.159] => (item={u'package': u'httpd'})
changed: [10.100.0.159] => (item={u'package': u'mod_ssl'})

PLAY RECAP *******************************************************************************************************
10.100.0.159               : ok=2    changed=1    unreachable=0    failed=0

Thursday 20 December 2018  08:47:36 +0000 (0:00:06.258)       0:00:07.113 *****
===============================================================================
httpd : install httpd with yum ---------------------------------------------------------------------------- 6.26s
Gathering Facts ------------------------------------------------------------------------------------------- 0.78s
```
Apache Webserver 를 설치하는 Task `install httpd with yum` 의 실행 결과를 보면, `changed` 로 변경사항이 적용된 것을 볼 수 있습니다.

### 같은 내용 다시 실행하기
이미 실행한 script 이지만 target server에 한번 더 실행을 해보도록 합니다. <br>
```bash
# 같은 내용을 2번째 실행
$ ansible-playbook -l target playbook/httpd.yml
```
```txt
PLAY [all] *******************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************
Thursday 20 December 2018  08:52:12 +0000 (0:00:00.063)       0:00:00.063 *****
ok: [10.100.0.159]

TASK [httpd : install httpd with yum] ****************************************************************************
Thursday 20 December 2018  08:52:12 +0000 (0:00:00.720)       0:00:00.783 *****
ok: [10.100.0.159] => (item={u'package': u'httpd'})
ok: [10.100.0.159] => (item={u'package': u'mod_ssl'})

PLAY RECAP *******************************************************************************************************
10.100.0.159               : ok=2    changed=0    unreachable=0    failed=0

Thursday 20 December 2018  08:52:13 +0000 (0:00:00.772)       0:00:01.556 *****
===============================================================================
httpd : install httpd with yum ---------------------------------------------------------------------------- 0.77s
Gathering Facts ------------------------------------------------------------------------------------------- 0.72s
```
이 번에는 결과가 `ok`로 변경사항이 없습니다. <br>
해당 package에 변경이 없기 때문에 Ansible은 변경이 없는 것으로 판단합니다.

같은 Ansible script를 2번 이상 수행 하더라도 결과는 같아야 합니다.<br>
Ansible 에서는 같은 스크립트가 여러번 수행되도 같은 형상의 결과가 나오는 것을 멱등성(idempotence)라고 표현하고 있습니다.

만약 여러분이 작성한 Ansible script를 수행할 때마다 `changed`로 결과가 나온다면, 형상이 바뀌기 때문에 어떤 문제가 있는지 판단을 해보아야 합니다. <br>
실제 production 환경에서 모든 script가 멱등성을 유지할 수 있다면 좋겠지만, 환경에 따라 멱등성을 유지하기 어려운 경우도 간혹 있습니다. <br>
여러분들은 최대한 멱등성을 유지할 수 있도록 Ansible을 여러번 테스트 하시는게 중요합니다. <br>

이제 다음 단계로 넘어가 변수처리에 대해 연습을 합니다.
