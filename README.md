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
docker build -t django_app:v1 .
```

Проверить добавился ли образ в Minikube:

```
minikube image ls
```

Создайте файл .env в корневой директории проекта и добавьте переменные окружения:

- `DEBUG` — дебаг-режим
- `DATABASE_URL` — Данные для базы данных PostgreSQL в формате postgres://USER:PASSWORD@HOST:PORT/NAME
- `ALLOWED_HOSTS` — [см. документацию Django](https://docs.djangoproject.com/en/3.1/ref/settings/#allowed-hosts)

Создайте ConfigMap в Kubernetes:
```
kubectl create configmap django-config --from-file=.env
```
Создайте манифесты Deployment и Service:
```
kubectl apply -f deployment.yaml -f django-service.yaml
```

## Примечания

После изменения переменных окружения необходимо перезапустить под:
```
kubectl delete configmap django-config
kubectl create configmap django-config --from-file=.env
kubectl rollout restart deployment django
```