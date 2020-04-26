Kubernetes
=========

Requirements
------------

Три сервера с установленной операционной системой CentOS 7.

Не менее двух логических процессоров на каждом.

Если не используется свой dns-сервер, то необходимо прописать их адреса в файле hosts на всех серверах.

Role Variables
--------------

```
user_name: centos # имя пользователя из под которого в дальнейшем будут запускаться плейбуки
```

Пример файла инвентаря hosts (не путать с системным файлом hosts)
------------

```
master ansible_host=192.168.122.10
node1 ansible_host=192.168.122.11
node2 ansible_host=192.168.122.12


[kube_cluster]
master
node1
node2

[kube_nodes]
node1
node2
```

Example Playbook
----------------

```
---
# Устанавлием компоненты, необходимые на всех нодах
- hosts: kube_cluster
  roles:
    - { role: kuber }

# Инициализируем master-ноду
- hosts: master
  vars_files: roles/kuber/vars/main.yml
  tasks:
    - name: initialize the master
      include: roles/kuber/tasks/init-master.yml

# Присоединяем рабочие ноды
- hosts: kube_nodes
  tasks:
    - name: join cluster
      shell: "{{ hostvars['master'].join_command }} --ignore-preflight-errors all  >> node_joined.txt"
      args:
        chdir: $HOME
        creates: node_joined.txt
```

Подключение к WebUI
-------------------
Так как Dashbaord работает исключительно на localhost, перед подключением нужно пробросить порт до вебморды
```
ssh -L 8001:127.0.0.1:8001 user@ip_or_hostname_master
```
После чего в браузере:

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
