# Django site in Minikube

Докеризированный сайт на Django для экспериментов с Kubernetes.


## Как запустить

Перейти в окружение Docker в Minikube:

```
eval $(minikube docker-env)
```

Перейдите в каталог с Docker-образом:

```
cd k8s-test-django/backend_main_django
```

Соберите Docker-образ:

```
docker build -t django_app:v88 .
```

Проверить добавился ли образ в Minikube:

```
minikube image ls
```

Создайте файл .env в корневой директории проекта и добавьте переменные окружения:

- `DEBUG` — дебаг-режим
- `DATABASE_URL` — Данные для базы данных PostgreSQL в формате postgres://USER:PASSWORD@HOST:PORT/NAME
- `ALLOWED_HOSTS` — [см. документацию Django](https://docs.djangoproject.com/en/3.1/ref/settings/#allowed-hosts)

Установите [Helm](https://helm.sh/)

Разверните PostgreSQL в кластере:
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
```
helm install postgres bitnami/postgresql
```
В качестве хоста БД используйте
```
postgres.default.svc.cluster.local
```
Добавьте его в DATABASE_URL .env файла


Сохраните пароль в переменной окружения POSTGRES_PASSWORD
```
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgres-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
```
Чтобы подключиться к вашей базе данных, выполните следующую команду:
```
kubectl run postgres-postgresql-client --rm --tty -i --restart='Never' --namespace default --image docker.io/bitnami/postgresql:16.1.0-debian-11-r20 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgres-postgresql -U postgres -d postgres -p 5432
```
Создайте БД и пользователя в PostgreSQL:

Данные для БД возьмите из переменной DATABASE_URL в .env файле
```
create database Имя_вашей_бд;
create user ваш_логин with encrypted password 'пароль_пользователя';
grant all privileges on database Имя_вашей_бд to ваш_логин;
ALTER DATABASE Имя_вашей_бд OWNER TO ваш_логин;
```
Получите IP адрес из команды kubectl cluster-info
```
Добавить в конец
Например 192.168.49.2 star-burger.test
В файл
/etc/hosts
```

Создайте ConfigMap в Kubernetes:
```
kubectl create configmap django-config --from-file=/путь до файла/.env
(Укажите полный путь до .env файла /opt/project/.env)
```
Создайте манифесты Deployment, Service, ingress, Jobs для очистки сессий и миграций :
```
kubectl apply -f deployment.yaml -f django-service.yaml -f ingress.yaml -f django-clearsessions.yaml -f migrate-job.yaml 
```

## Примечания

После изменения переменных окружения необходимо перезапустить под:
```
kubectl delete configmap django-config
kubectl create configmap django-config --from-file=/путь до файла/.env
kubectl rollout restart deployment django
```