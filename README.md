# Ansible
Первые шаги с Ansible
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере используя Ansible необходимо развернуть nginx со следующими условиями:
- необходимо использовать модуль yum/apt
- конфигурационные файлы должны быть взяты из шаблона jinja2 с перемененными
- после установки nginx должен быть в режиме enabled в systemd
- должен быть использован notify для старта nginx после установки
- сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible



Запустим новую виртуалку согласно vagrant file.пока без применения роли.

<code>vagrant up --no-provision </code>

Проверим конфиг ssh

```ruby
nginx vagrant ssh-config
Host nginx
  HostName 127.0.0.1
  User vagrant
  Port 2202
  UserKnownHostsFile /dev/null
  StrictHostKeyChecking no
  PasswordAuthentication no
  IdentityFile /root/nginx/.vagrant/machines/nginx/virtualbox/private_key
  IdentitiesOnly yes
````
создадим свой первый inventory файл
```ruby
nginx cat /etc/ansible/hosts

[web]
nginx ansible_host=127.0.0.1 ansible_port=2202 ansible_user=vagrant ansible_private_key_file=.vagrant/machines/nginx/virtualbox/private_key

```
убедимся, что Ansible может управлять нашим хостом. 
```ruby
➜  nginx ansible all -m ping

nginx | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

 Посмотрим какое ядро установлено на хосте:
  ```ruby
  nginx ansible nginx -m command -a "uname -r"
nginx | CHANGED | rc=0 >>
3.10.0-1127.el7.x86_64
```
Когда все подключения настроены запустим нашу роль
```ruby 
nginx ansible-playbook nginx.yml

PLAY [Install and configure NGINX] *******************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************
ok: [nginx]

TASK [Install EPEL Repo package from standart repo] **************************************************************************
ok: [nginx]

TASK [Install NGINX package from epel-repo] **********************************************************************************
ok: [nginx]

TASK [Create NGINX config file from template] ********************************************************************************
changed: [nginx]

RUNNING HANDLER [reload nginx] ***********************************************************************************************
changed: [nginx]

PLAY RECAP *******************************************************************************************************************
nginx                      : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Проверяем

```ruby
 nginx curl http://192.168.11.150:8080
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css">
  ```
