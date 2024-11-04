# Лабораторная №3

## Цель работы:
Поднять приложение на Django, используя Nginx для обработки статических файлов и прокси используя роли, научиться шаблонизации конфигураций.

Django-приложение: репо

Docker-образ: https://hub.docker.com/repository/docker/timurbabs/django

## Ход работы:
1.	Поднять Vagrant-окружение при помощи файла `Vagrantfile`, используя команду `vagrant up`
2.	Написать роль для установки nginx, запушить в репозиторий и добавить в `requirements.yml`
3.	Подготовить в инвентори шаблон конфигурационного файла для Nginx и для default-сайта, с настройками для отдачи статических файлов
4.	Учесть в шаблоне проксирование динамики в контейнер с Django
5.	Написать плейбук установки через роли nginx на хосты группы web, и docker на хосты группы app
Запустить приложение на хостах группы `[app]` или подготовить `docker-compose.yml` для запуска приложения

1.	Поднять Vagrant-окружение при помощи файла `Vagrantfile`, используя команду `vagrant up`

Выполним команду:
```
vagrant up
```
<img width="1249" alt="Снимок экрана 2024-11-04 в 18 17 03" src="https://github.com/user-attachments/assets/3c23fa54-b315-4a59-ba28-9db085abd74c">


2.	Написать роль для установки nginx, запушить в репозиторий и добавить в `requirements.yml`

Создаем роль Nginx - в директории `/ansible` при помощи команды:
```
ansible-galaxy init nginx
```
<img width="914" alt="Снимок экрана 2024-11-04 в 18 18 18" src="https://github.com/user-attachments/assets/66558097-7afd-43fc-8adb-5a020142003a">

Создаем задачи для роли Nginx в файле `/nginx/tasks/main.yml`:
```
- name: Install nginx
  apt:
    name: nginx
    state: present

- name: Copy nginx site configuration
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/default

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
```

3.	Подготовить в инвентори шаблон конфигурационного файла для Nginx и для default-сайта, с настройками для отдачи статических файлов (Учитывая в шаблоне проксирование динамики в контейнер с Django)

Создаем шаблон конфигурации `(/nginx/templates/nginx.conf.j2)`:
```
server {
    listen 80;
    server_name localhost;

    location /static/ {
        alias /path/to/static/files;    
}

    location / {
        proxy_pass http://localhost:8000; # Проксирование запросов к Django
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Cоздаем репозиторий в gitlab под названием `ansible-nginx-role`, переходим внутрь директории:
```
cd nginx
```
Инициадизируем новый git-репозиторий в текущем каталоге и добавляем новые файлы:
```
git init
git add .
git commit -m "Initial commit of nginx role"
```

<img width="1257" alt="Снимок экрана 2024-11-04 в 18 22 25" src="https://github.com/user-attachments/assets/9631c844-92cf-4286-b208-2a1de6640201">

Добавляем удаленный репозиторий и пушим изменения на него:
```
git remote add origin https://gitlab.com/andrey1707/ansible-nginx-role.git
git push -u origin master
```
<img width="1277" alt="Снимок экрана 2024-11-04 в 19 02 39" src="https://github.com/user-attachments/assets/38f8f6b8-f4ae-46c7-8384-4757f8adc610">

Обновляем файл `requirements.yml`
```
- src: https://gitlab.com/andrey1707/ansible-docker-role.git
  scm: git
  version: master
  name: Docker

- src: https://gitlab.com/andrey1707/ansible-nginx-role.git
  scm: git
  version: master
  name: nginx
```

4.	Написать плейбук установки через роли nginx на хосты группы web, и docker на хосты группы `[app]`

Создаем Playbook `(site.yml)` в папке `/ansible`.

Файл `site.yml`:

<img width="421" alt="Снимок экрана 2024-11-04 в 19 05 22" src="https://github.com/user-attachments/assets/6b841135-0ff8-40ec-b8d2-359597132fe1">

Файл `inventory.yml`:

<img width="438" alt="Снимок экрана 2024-11-04 в 19 04 26" src="https://github.com/user-attachments/assets/d2a724e9-0edf-4840-b4a1-d948b536c87a">


Запускаем playbook из директории `/ansible` при помощи команды:

```
ansible-playbook -i inventory.ini site.yml
```
<img width="1250" alt="Снимок экрана 2024-11-04 в 19 06 23" src="https://github.com/user-attachments/assets/ff330e9b-ff07-4abb-b785-5701e01bef98">





