ANSIBLE

В ansible после установки самое главное это создание двух файлов (необязательно, но для адекватного использования это надо, но можно ключи и просто так прописывать под каждый сервер) 

Эти файлы:1. Inventory-файл hosts.txt - можем навать как угодно 
             Так называемый inventory файл- инвентарь нашего ansible. В нем содержатся все                    адреса хостов и переменные для подлкючения к ним (и заданные нами переменные) 
             + в нем мы можем обьединять хосты в группы, группы в группы...
            
Пример:hosts.txt
```
[test_DB]

[test_APP]

[test_WEB]

[test_ALL:children]                 ПРИМЕР ОБЬЕДИНЕНИЯ ГРУПП В ГРУППЫ 
test_DB
test_APP
test_WEB
[servers_ALL:children]
servers
[servers]
server1   ansible_host=192.168.31.201   ansible_user=server1
server2   ansible_host=192.168.31.42    ansible_user=server2
[servers:vars]  ЕСЛИ У СЕРВЕРОВ В ГРУППЕ ЕСТЬ ОБЩИЕ ПЕРЕМЕННЫЕ, МОЖЕМ ИХ ОБЬЕДИНИТЬ ЧТОБЫ НЕ ПРОПИСЫВАТЬ КАЖДЫЙ РАЗ + НАГЛЯДНО ВЫГЛЯДИТ ЛУЧШЕ

ansible_ssh_password=password

```

 2. ansible.cfg -конфиг файл ansible, в нем мы можем указать различные параметры,
 САМОЕ ВАЖНОЕ- указать наш inventory файл, чтобы каждый раз не 
 припысывать ansible -i config.txt + можно указать всякие ключи чтобы их также каждый            раз не писать.

Пример: ansible.cfg 
```
[defaults]
host_key_checking = false  #-убирает чек фингерпринта, дает возможность заходить просто по паролю без ssh key#
inventory         = hosts.txt   #-указываем наш inventory файл

```

Some interesting commands:
ansible --version
ansible-inventory --list  - список всех серверов групп и тд
ansible-inventory --graph - граф из серверов по группам 

Модули Ansible 
https://docs.ansible.com/ansible/2.9/user_guide/modules.html
Просмотр всех мудулей:ansible-doc -l
Для запуска модуля: ansible группа -m название модуля        Пример:ansible all -m ping
-m ping 
-m setup - инфа о системе 
-m shell -a "uptime"  -m shell -a "ls /etc" - запуск команд через shell , работает ВСЕ
-m command -a "ls"    -аналог модуля shell, не не работают пайпы, потоки ввода- вывода , только одна команда, также не работают внтуренние переменные

Модуль copy
ansible all -m copy -a "src=privet.txt dest=/home mode=777" -b 
src=  - указываем source к файлу 
dest= - указываем destination к файлу 
mode= - указываем права доступа к файлу( можно не указывать) 
-b -запускаем с sudo b- become sudo user ТАК КАК В ПАПКУ HOME без судо ничего не засунуть 
Для использования -b в extra_vars в inventory файле надо указать ansible_sudo_pass= пароль

Модуль file 
Много разных возможностей, в примере удаляем файл, надо указать путь до файла и его state  state=absent - отсутствует 
ansible all -m file -a "path=/home/privet.txt state=absent"  -b

Модуль get_url
Скачать какой нибудь файл по ссылке указываем юрл и дест
ansible all -m get_url -a "url= FILE URL dest=/home/......" -b 

Модуль apt/yum/package
Установка приложений
ansible all -m apt -a "name=stress state=latest" -b
Модуль apt под Ubuntu
Модуль yum под centOS
Также есть модуль package под убунту 

state=latest - установка последней версии 
state=removed - удаление для yum 
state=absent удаление для apt 

Модуль uri 
Проверка доступа к тому или иному сайту, return_content- вывод как в curl
ansible all -m uri -a "url= http://google.com return_content=yes"

Модуль service
Работа с сервисами 
ansible all -m service -a "name=httpd state=started enabled=yes" -b 
state=started - запуск приложение 
enabled=yes   - автозапуск при старте

ДЕБАГГИНГ
ДЛЯ ДЕБАГГИНГА В КОНЕЦ КОМАНДЫ ДОПИСЫВАЕМ -v -vv -vvv -vvvv -vvvvv 
Количество v определяет подробность и обьем информации на выводе

Плэйбуки PLAYBOOKS
Плэйбуки ansible весьма просты в понимании, главное разобраться в yaml формате и логике построения. По сути в плейбуках учавствуют теже самые модули с такими же аргументами.
Плэйбуки позволяют использовать блоки модулей, выстраивать между ними логику, также добавлять  переменные (например различные директории, сообщения и т.д.)
ЗАПУСК ПЛЕЙБУКА 
ansible-playbook (название.yml)

Примеры плэйбуков
1.
---  (начало yml файла)
- name: Test connection to server 
  hosts: all
  become: yes 
  
  tasks:
  - name: ping my servers 
  ping:
  
  
  #####
  Просто пингуем наши сервера, которые мы до этого указали в inventory
  #####
 2.
 
---
- name: Install nginx  web server
  hosts: all
  become: yes


  tasks:
  - name: Install nginx
    apt:  name=nginx state=latest

  - name: Start service nginx and enable it on every boot
    service: name=nginx state=started enabled=yes
    
  #####
  Устанавливаем на сервера nginx, по сути все тоже самое что и с подулем apt только чуток в       другом формате 
  #####
  
  Дальше все фичи использования плэйбуков демонстрируются в yaml файлах в моем репозитории








         
         
         
