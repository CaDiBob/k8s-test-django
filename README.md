# Django site

Докеризированный сайт на Django для экспериментов с Kubernetes.

Внутри конейнера Django запускается с помощью Nginx Unit, не путать с Nginx. Сервер Nginx Unit выполняет сразу две функции: как веб-сервер он раздаёт файлы статики и медиа, а в роли сервера-приложений он запускает Python и Django. Таким образом Nginx Unit заменяет собой связку из двух сервисов Nginx и Gunicorn/uWSGI. [Подробнее про Nginx Unit](https://unit.nginx.org/).

## Как запустить dev-версию

Запустите базу данных и сайт:

```shell-session
$ docker-compose up
```

В новом терминале не выключая сайт запустите команды для настройки базы данных:

```shell-session
$ docker-compose run web ./manage.py migrate  # создаём/обновляем таблицы в БД
$ docker-compose run web ./manage.py createsuperuser
```

Для тонкой настройки Docker Compose используйте переменные окружения. Их названия отличаются от тех, что задаёт docker-образа, сделано это чтобы избежать конфликта имён. Внутри docker-compose.yaml настраиваются сразу несколько образов, у каждого свои переменные окружения, и поэтому их названия могут случайно пересечься. Чтобы не было конфликтов к названиям переменных окружения добавлены префиксы по названию сервиса. Список доступных переменных можно найти внутри файла [`docker-compose.yml`](./docker-compose.yml).

## Как запустить в кластере Minikube

[Docker](https://docs.docker.com/engine/install/), [Minikube](https://kubernetes.io/ru/docs/tasks/tools/install-minikube/), [kubectl](https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/) должны быть установлены.

##### Склонировать проект:

```bash
git clone git@github.com:CaDiBob/k8s-test-django.git
```

##### Собрать Docker-образ:

```bash
sudo docker build -t ваше_имя_на_докерхаб/имя_репозитория backend_main_django/
```
##### Запушить в ваш докерхаб:

```bash
sudo docker push ваше_имя_на_докерхаб/имя_репозитория
```
##### Запускаем локальный кластер Minikube:

```bash
minikube start
```

##### Запускаем ingress:

```bash
minikube addons enable ingress
```

##### Создаем пространство имен для нашего проекта:

```bash
kubectl create ns somename
```
Дальше все команды будем выполнять в созданном пространстве имен.

##### Запустим наш проект в кластере Minikube:

```bash
kubectl -n somename apply -f manifests/
```
Сайт будет доступен по адресу cadibob.test для этого нужно добавить ваш IP Minikube в файл hosts.

```bash
minikube ip
```

##### Чтобы завершить работу проекта:

```bash
kubectl -n somename delete -f manifests/
```

##### Добавление переменных окружения:

В файле ex-configmap.yaml прописать свои значения переменных и переименовать в configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: example
data:
  SECRET_KEY: "Ваш SECRET_KEY проекта Django"
  DEBUG: "False"
  DATABASE_URL: "postgres://имя пользователя БД:пароль пользователя БД@адрес хоста:порт/имя БД"
  ALLOWED_HOSTS: localhost,127.0.0.1
```

## Переменные окружения

Образ с Django считывает настройки из переменных окружения:

`SECRET_KEY` -- обязательная секретная настройка Django. Это соль для генерации хэшей. Значение может быть любым, важно лишь, чтобы оно никому не было известно. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#secret-key).

`DEBUG` -- настройка Django для включения отладочного режима. Принимает значения `TRUE` или `FALSE`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#std:setting-DEBUG).

`ALLOWED_HOSTS` -- настройка Django со списком разрешённых адресов. Если запрос прилетит на другой адрес, то сайт ответит ошибкой 400. Можно перечислить несколько адресов через запятую, например `127.0.0.1,192.168.0.1,site.test`. [Документация Django](https://docs.djangoproject.com/en/3.2/ref/settings/#allowed-hosts).

`DATABASE_URL` -- адрес для подключения к базе данных PostgreSQL. Другие СУБД сайт не поддерживает. [Формат записи](https://github.com/jacobian/dj-database-url#url-schema).
