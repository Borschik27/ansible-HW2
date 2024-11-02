# ansible-HW2
Задание 1 ClickHouse

```
sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook$ docker build -t ubuntu -f playbook/answer/dockerfile.ubuntu playbook/answer/.
[+] Building 3.2s (6/6) FINISHED                                                                         docker:default
 => [internal] load build definition from dockerfile.ubuntu                                                        0.0s
 => => transferring dockerfile: 463B                                                                               0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                                                    3.1s
 => [internal] load .dockerignore                                                                                  0.0s
 => => transferring context: 2B                                                                                    0.0s
 => [1/2] FROM docker.io/library/ubuntu:20.04@sha256:8e5c4f0285ecbb4ead070431d29b576a530d3166df73ec44affc1cd27555  0.0s
 => CACHED [2/2] RUN apt-get update &&     apt-get install -y python3 python3-pip &&     apt-get clean             0.0s
 => exporting to image                                                                                             0.0s
 => => exporting layers                                                                                            0.0s
 => => writing image sha256:a0a6cae374b1926ced510fca71c5d4641f7d12e44ba6df7b2b9e4290ce2aadd1                       0.0s
 => => naming to docker.io/library/ubuntu                    
```

```
sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ docker run -d --name clickhouse-01 ubuntu:latest
75af0c2dd0905bd3f551a38540a0589d84f6afe33c4768fe51cd6c428612a705
```

```
sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site.yml

PLAY [Install Clickhouse] **********************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [clickhouse-01]

TASK [Install dependencies] ********************************************************************************************
changed: [clickhouse-01]

TASK [Download and add Clickhouse GPG key] *****************************************************************************
changed: [clickhouse-01]

TASK [Add Clickhouse repository] ***************************************************************************************
changed: [clickhouse-01]

TASK [Update apt cache] ************************************************************************************************
ok: [clickhouse-01]

TASK [Install clickhouse packages] *************************************************************************************
changed: [clickhouse-01]

TASK [Flush handlers] **************************************************************************************************

RUNNING HANDLER [Start clickhouse service] *****************************************************************************
changed: [clickhouse-01]

TASK [Create database] *************************************************************************************************
FAILED - RETRYING: [clickhouse-01]: Create database (5 retries left).
FAILED - RETRYING: [clickhouse-01]: Create database (4 retries left).
FAILED - RETRYING: [clickhouse-01]: Create database (3 retries left).
FAILED - RETRYING: [clickhouse-01]: Create database (2 retries left).
FAILED - RETRYING: [clickhouse-01]: Create database (1 retries left).
fatal: [clickhouse-01]: FAILED! => {"attempts": 5, "changed": false, "cmd": ["clickhouse-client", "-q", "create database logs;"], "delta": "0:00:00.031636", "end": "2024-11-02 12:31:18.509615", "failed_when_result": true, "msg": "non-zero return code", "rc": 210, "start": "2024-11-02 12:31:18.477979", "stderr": "Code: 210. DB::NetException: Connection refused (localhost:9000). (NETWORK_ERROR)", "stderr_lines": ["Code: 210. DB::NetException: Connection refused (localhost:9000). (NETWORK_ERROR)"], "stdout": "", "stdout_lines": []}

PLAY RECAP *************************************************************************************************************
clickhouse-01              : ok=7    changed=5    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0

sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$
```

Ошибка возникает из-за того что в сборке контейнера недостает пакетов для запуска исправляю это в самом контейнере

```
root@30c002dea999:/# service clickhouse-server start
 chown -R clickhouse: '/var/run/clickhouse-server/'
Will run sudo --preserve-env -u 'clickhouse' /usr/bin/clickhouse-server --config-file /etc/clickhouse-server/config.xml --pid-file /var/run/clickhouse-server/clickhouse-server.pid --daemon
/bin/sh: 1: sudo: not found
Code: 302. DB::Exception: Child process was exited with return code 127. (CHILD_WAS_NOT_EXITED_NORMALLY) (version 24.10.1.2812 (official build))
root@30c002dea999:/# apt-get update
Hit:1 http://security.ubuntu.com/ubuntu focal-security InRelease
Hit:2 https://packages.clickhouse.com/deb stable InRelease
Hit:3 http://archive.ubuntu.com/ubuntu focal InRelease
Hit:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease
Hit:5 http://archive.ubuntu.com/ubuntu focal-backports InRelease
Reading package lists... Done
root@30c002dea999:/# apt-get install sudo
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
  sudo
0 upgraded, 1 newly installed, 0 to remove and 3 not upgraded.
Need to get 515 kB of archives.
After this operation, 2257 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 sudo amd64 1.8.31-1ubuntu1.5 [515 kB]
Fetched 515 kB in 1s (405 kB/s)
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package sudo.
(Reading database ... 14628 files and directories currently installed.)
Preparing to unpack .../sudo_1.8.31-1ubuntu1.5_amd64.deb ...
Unpacking sudo (1.8.31-1ubuntu1.5) ...
Setting up sudo (1.8.31-1ubuntu1.5) ...
root@30c002dea999:/# sudo service clickhouse-server start
 chown -R clickhouse: '/var/run/clickhouse-server/'
Will run sudo --preserve-env -u 'clickhouse' /usr/bin/clickhouse-server --config-file /etc/clickhouse-server/config.xml --pid-file /var/run/clickhouse-server/clickhouse-server.pid --daemon
Waiting for server to start
Waiting for server to start
Server started
root@30c002dea999:/# sudo service clickhouse-server status
/var/run/clickhouse-server/clickhouse-server.pid file exists and contains pid = 2559.
The process with pid = 2559 is running.

root@30c002dea999:/# clickhouse-client -q "SELECT version()"
24.10.1.2812
```

Задача 2 Vector

```
sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site_vector.yml

PLAY [Install Vector] **************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [vector-01]

TASK [Install dependencies] ********************************************************************************************
changed: [vector-01]

TASK [Download Vector deb package] *************************************************************************************
changed: [vector-01]

TASK [Install Vector] **************************************************************************************************
changed: [vector-01]

TASK [Deploy Vector configuration] *************************************************************************************
changed: [vector-01]

RUNNING HANDLER [Restart Vector] ***************************************************************************************
changed: [vector-01]

PLAY RECAP *************************************************************************************************************
vector-01                  : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
Скрины что запущен vector И применне конфиг из шаблона

![image](https://github.com/user-attachments/assets/d75e465a-2519-4bd4-bd60-d5a3e12f3f5e)

![image-1](https://github.com/user-attachments/assets/7f80182d-b168-41e9-9800-b6d28c2d0269)

Дальше я выполнилл --check --diff не внося изменений 
После чего внес изменени в шаблон и повторил --check --diff
```
sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site_vector.yml --check

PLAY [Install Vector] **************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [vector-01]

TASK [Install dependencies] ********************************************************************************************
ok: [vector-01]

TASK [Download Vector deb package] *************************************************************************************
ok: [vector-01]

TASK [Install Vector] **************************************************************************************************
ok: [vector-01]

TASK [Deploy Vector configuration] *************************************************************************************
ok: [vector-01]

PLAY RECAP *************************************************************************************************************
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site_vector.yml --diff

PLAY [Install Vector] **************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [vector-01]

TASK [Install dependencies] ********************************************************************************************
ok: [vector-01]

TASK [Download Vector deb package] *************************************************************************************
ok: [vector-01]

TASK [Install Vector] **************************************************************************************************
ok: [vector-01]

TASK [Deploy Vector configuration] *************************************************************************************
ok: [vector-01]

PLAY RECAP *************************************************************************************************************
vector-01                  : ok=5    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site_vector.yml --check

PLAY [Install Vector] **************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [vector-01]

TASK [Install dependencies] ********************************************************************************************
ok: [vector-01]

TASK [Download Vector deb package] *************************************************************************************
ok: [vector-01]

TASK [Install Vector] **************************************************************************************************
ok: [vector-01]

TASK [Deploy Vector configuration] *************************************************************************************
changed: [vector-01]

RUNNING HANDLER [Restart Vector] ***************************************************************************************
changed: [vector-01]

PLAY RECAP *************************************************************************************************************
vector-01                  : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

sypchik@Mirror:/mnt/c/Users/Sypchik/Desktop/home work/ansible/mnt-homeworks/08-ansible-02-playbook/playbook$ ansible-playbook -i inventory/prod.yml site_vector.yml --diff

PLAY [Install Vector] **************************************************************************************************

TASK [Gathering Facts] *************************************************************************************************
ok: [vector-01]

TASK [Install dependencies] ********************************************************************************************
ok: [vector-01]

TASK [Download Vector deb package] *************************************************************************************
ok: [vector-01]

TASK [Install Vector] **************************************************************************************************
ok: [vector-01]

TASK [Deploy Vector configuration] *************************************************************************************
--- before: /etc/vector/vector.yaml
+++ after: /home/sypchik/.ansible/tmp/ansible-local-21780qi_kk8l0/tmpmxoz7ri8/vector_config.j2
@@ -1,4 +1,4 @@
-# Custom Sypchik Configuration
+# Custom Sypchik Configuration 22
 #                                    __   __  __
 #                                    \ \ / / / /
 #                                     \ V / / /

changed: [vector-01]

RUNNING HANDLER [Restart Vector] ***************************************************************************************
changed: [vector-01]

PLAY RECAP *************************************************************************************************************
vector-01                  : ok=6    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```
