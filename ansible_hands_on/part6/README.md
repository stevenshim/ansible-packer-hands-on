
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
