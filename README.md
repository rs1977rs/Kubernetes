Kubernetes
=========

Requirements
------------

Три сервера с установленной операционной системой CentOS 7.

Если не используется свой dns-сервер, то необходимо прописать их адреса в файле hosts на всех серверах.

Role Variables
--------------

```
user_name: centos # имя пользователя из под которого в дальнейшем будут запускаться плейбуки
```

Пример, как может выглядеть файл hosts (не путать с системным файлом hosts)
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
