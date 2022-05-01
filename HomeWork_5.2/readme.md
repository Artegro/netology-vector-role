# Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами"

## Задача 1

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.
```
Основное преимущество , более быстрое развертывание необходимых контейнеров, инфраструктур,
сервисов с предсказуемым результатом, в силу того, что у нас вся инфраструктура описана кодом,
мы можем создавать много одинаковых объектов, изменив код,  можем влиять на все объекты или на часть.
```
- Какой из принципов IaaC является основополагающим?
```
Основополагающий принцип , как я понял как раз и является само название инфраструктура как код,
то есть это описание всех установок настроек систем кодом.
```
## Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?
```
Если прям честно, то тем, что его здесь преподают.
А если серьёзно, то как я могу сказать чем он лучше не имея опыта использования его или другого ПО.
Если верить лекции, то Ansible самый простой и инструментов , так же использует 2 подхода Декларативный и Императивный и метод распространения Push.
```
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?
```
Сложный вопрос, с одной стороны лучше Push, когда мы сами инициируем процес с управлящего сервера (мы кантролируем),
с другой мы же сами настраиваем Pull когда и как клиентым получать, то есть опять таки контроль у нас.
По сути метод Push лучше, когда у нас неизвесно заранее, когда будут обновленния,
а метод Pull лучше, когда у нас обновления идут планово , раз в неделю например.
(хотя что нам мешает в случае Push сделать в крон нужную запись на управляющем сервере)
И выходит Push таки лучше, боле гибкий, что ли.
Про надежность, то тут у нас я думаю паритет , так как надежность доставки больше зависит от сети,
а не от того, кто инициировал процесс получения конфигурации.
Разница будет лишь в том, что при Push обнонятся только те клиенты , до которых достучиться управляющей сервер,
а при Pull обновятся только те клиенты, которые смогут достучаться.
Но так как мы же настроили мониторинг и знаем кто обновился а кто нет, то нам по сути не так важен сам сметод, как конечный результат.
Так же при Push мы можем вибирать время минимальной нагрузки на управляющий сервер , чтоб не положить его одновремнным обновлением всех клиентов.
Выходит Push немного надежнее в больших инфраструктурах.
```


## Задача 3

Установить на личный компьютер:
### VirtualBox
```bash
#  virtualbox -h
Oracle VM VirtualBox VM Selector v6.1.32_Debian
(C) 2005-2022 Oracle Corporation
All rights reserved.
```
### Vagrant
``` bash
# vagrant -v
Vagrant 2.2.14

# vagrant box add bento/ubuntu-20.04 --provider=virtualbox --force
==> box: Loading metadata for box 'bento/ubuntu-20.04'
    box: URL: https://vagrantcloud.com/bento/ubuntu-20.04
==> box: Adding box 'bento/ubuntu-20.04' (v202112.19.0) for provider: virtualbox
    box: Downloading: https://vagrantcloud.com/bento/boxes/ubuntu-20.04/versions/202112.19.0/providers/virtualbox.box
Download redirected to host: vagrantcloud-files-production.s3-accelerate.amazonaws.com
==> box: Successfully added box 'bento/ubuntu-20.04' (v202112.19.0) for 'virtualbox'!

# vagrant box list
bento/ubuntu-20.04 (virtualbox, 202112.19.0)
```
- Старт вм , с конфигом из лекции , проверка Vagrant
``` bash
# vagrant status
Current machine states:

server1.netology          running (virtualbox)
# vagrant ssh
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 1.0


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

vagrant@server1:~$ cat /etc/*release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.3 LTS"
NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.3 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```
### Ansible
``` bash
# ansible --version
ansible 2.10.8
  config file = None
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.9.2 (default, Feb 28 2021, 17:03:44) [GCC 10.2.1 20210110]
```


## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

- Создать виртуальную машину.
- Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
docker ps
```
### Ответ:
- Виртульная машина создана . конфики взяты полностью из лекции, несмотря на ошибку ssh Key в процессе создания вм , установка докеар отработала
```bash
# vagrant reload --provision
==> server1.netology: Attempting graceful shutdown of VM...
==> server1.netology: Checking if box 'bento/ubuntu-20.04' version '202112.19.0' is up to date...
==> server1.netology: Clearing any previously set forwarded ports...
==> server1.netology: Clearing any previously set network interfaces...
==> server1.netology: Preparing network interfaces based on configuration...
    server1.netology: Adapter 1: nat
    server1.netology: Adapter 2: hostonly
==> server1.netology: Forwarding ports...
    server1.netology: 22 (guest) => 20011 (host) (adapter 1)
    server1.netology: 22 (guest) => 2222 (host) (adapter 1)
==> server1.netology: Running 'pre-boot' VM customizations...
==> server1.netology: Booting VM...
==> server1.netology: Waiting for machine to boot. This may take a few minutes...
    server1.netology: SSH address: 127.0.0.1:2222
    server1.netology: SSH username: vagrant
    server1.netology: SSH auth method: private key
==> server1.netology: Machine booted and ready!
==> server1.netology: Checking for guest additions in VM...
==> server1.netology: Setting hostname...
==> server1.netology: Configuring and enabling network interfaces...
==> server1.netology: Mounting shared folders...
    server1.netology: /vagrant => /home/artegro/vagrantfile
==> server1.netology: Running provisioner: ansible...
    server1.netology: Running ansible-playbook...

PLAY [nodes] *******************************************************************

TASK [Gathering Facts] *********************************************************
ok: [server1.netology]

TASK [Create directory for ssh-keys] *******************************************
ok: [server1.netology]

TASK [Adding rsa-key in /root/.ssh/authorized_keys] ****************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: If you are using a module and expect the file to exist on the remote, see the remote_src option
fatal: [server1.netology]: FAILED! => {"changed": false, "msg": "Could not find or access '~/.ssh/id_rsa.pub' on the Ansible Controller.\nIf you are using a module and expect the file to exist on the remote, see the remote_src option"}
...ignoring

TASK [Checking DNS] ************************************************************
changed: [server1.netology]

TASK [Installing tools] ********************************************************
ok: [server1.netology] => (item=['git', 'curl'])

TASK [Installing docker] *******************************************************
changed: [server1.netology]

TASK [Add the current user to docker group] ************************************
changed: [server1.netology]

PLAY RECAP *********************************************************************
server1.netology           : ok=7    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=1

root@test:/home/artegro/vagrantfile# vagrant ssh
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun 01 May 2022 08:25:53 PM UTC

  System load:  0.98               Users logged in:          0
  Usage of /:   13.6% of 30.88GB   IPv4 address for docker0: 172.17.0.1
  Memory usage: 24%                IPv4 address for eth0:    10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1:    192.168.56.11
  Processes:    116


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Sun May  1 20:25:39 2022 from 10.0.2.2
vagrant@server1:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
vagrant@server1:~$
```


