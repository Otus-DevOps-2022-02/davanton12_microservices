# davanton12_microservices
davanton12 microservices repository

## Лекция 15-16

### Docker-1

Файл docker-1.log

### Docker-2

Docker хост в Yandex Cloud

Команды docker 
```
docker version
docker info
docker run hello-world
docker ps
docker ps -a
docker images
docker run -it ubuntu:18.04 /bin/bash
docker ps -a --format "table {{.ID}}\t{{.Image}}\t{{.CreatedAt}}\t{{.Names}}"
docker start <u_container_id>
docker attach <u_container_id>
docker run -dt nginx:latest
docker exec -it a5ca8a74eaa8 bash
docker commit a5ca8a74eaa8 adavidenko/nginx-tmp-file
docker images
docker inspect 7425d3a7c478
docker ps -q
docker kill $(sudo docker ps -q)
docker system df
docker rm $(sudo docker ps -a -q) удалит все незапущенные контейнеры
docker rmi $(sudo docker images -q)
```

Создадим Docker хост в Yandex Cloud
```
yc compute instance create \
 --name docker-host \
 --zone ru-central1-a \
 --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
 --create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
 --ssh-key ~/.ssh/appuser.pub
```

Смотрим какой ip-address
```
one_to_one_nat:
address: 51.250.86.254
ip_version: IPV4
```

Установка docker-machine
```
base=https://github.com/docker/machine/releases/download/v0.16.0 &&
curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
sudo mv /tmp/docker-machine /usr/local/bin/docker-machine &&
chmod +x /usr/local/bin/docker-machine
```

Создаем docker-machine
```
docker-machine create \
 --driver generic \
 --generic-ip-address=51.250.86.254 \
 --generic-ssh-user yc-user \
 --generic-ssh-key ~/.ssh/appuser \
 docker-host
 ```
 
Команды docker-machine
```
docker-machine ls
Команда создания - docker-machine create <имя>
Переключение между ними через eval $(docker-machine env <имя>)
Переключение на локальный докер - eval $(docker-machine env --unset)
Удаление - docker-machine rm <имя>
sudo docker run --rm -ti tehbilly/htop
sudo docker run --rm --pid host -ti tehbilly/htop
```

Сборка и запуск образа
```
docker build -t reddit:latest . - Собрать образ
docker images -a
docker run --name reddit -d --network=host reddit:latest
docker logs your_container_id
```
Работа с Docker.Hub
```
sudo docker login
docker tag reddit:latest adavidenko/otus-reddit:1.0
docker push adavidenko/otus-reddit:1.0
docker run --name reddit -d -p 9292:9292 adavidenko/otus-reddit:1.0
```
Проверка
```
docker logs reddit -f
docker exec -it reddit bash
 - ps aux
 - killall5 1
docker start reddit
docker stop reddit && docker rm reddit
docker run --name reddit --rm -it /otus-reddit:1.0 bash
 - ps aux
 - exit
docker inspect /otus-reddit:1.0
docker inspect /otus-reddit:1.0 -f '{{.ContainerConfig.Cmd}}'
docker run --name reddit -d -p 9292:9292 /otus-reddit:1.0

docker exec -it reddit bash
 - mkdir /test1234
 - touch /test1234/testfile
 - rmdir /opt
 - exit
docker diff reddit
docker stop reddit && docker rm reddit
docker run --name reddit --rm -it /otus-reddit:1.0 bash
 - ls /
```
Зачистка
```
docker-machine rm docker-host
yc compute instance delete docker-host
```

## Лекция 17

### Docker-3
Создадим Docker хост в Yandex Cloud
```
yc compute instance create
```
Установка docker-machine
```
docker-machine create
```
Смотрим
```
docker-machine ls
```
Переключаемся
```
eval $(docker-machine env docker-hosts)
```
Скачали образ
```
docker pull mongo:latest
```
Соберем образы с нашими сервисами:
 - сервис отвечающий за написание постов
```
docker build -t adavidenko/post:1.0 ./post-py
```
 - сервис отвечающий за написание комментариев
```
docker build -t adavidenko/comment:1.0 ./comment
```
 - веб-интерфейс, работающий с другими сервисами
```
docker build -t adavidenko/ui:1.0 ./ui
```
Создадим специальную сеть для приложения:
```
docker network create reddit
```
Запустим наши контейнеры
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db mongo:latest
docker run -d --network=reddit --network-alias=post adavidenko/post:1.0
docker run -d --network=reddit --network-alias=comment adavidenko/comment:1.0
docker run -d --network=reddit -p 9292:9292 adavidenko/ui:1.0
```
Проверка - http://51.250.90.59:9292

Остановка контейнеров
```
docker kill $(docker ps -q)
```
Поменял алиасы
```
docker run -d --network=reddit --network-alias=post_db_new --network-alias=comment_db_new mongo:latest
docker run -d --network=reddit --network-alias=post_new -e POST_DATABASE=posts_new -e POST_DATABASE_HOST=post_db_new  adavidenko/post:1.0
docker run -d --network=reddit --network-alias=comment_new -e COMMENT_DATABASE_HOST=comment_db_new -e COMMENT_DATABASE=comments_new adavidenko/comment:1.0
docker run -d --network=reddit -p 9292:9292 -e POST_SERVICE_HOST=post_new -e COMMENT_SERVICE_HOST=comment_new adavidenko/ui:1.0
```
Создадим Docker volume:
```
docker volume create reddit_db
```
Запуск с новыми параметрами
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post adavidenko/post:1.0
docker run -d --network=reddit --network-alias=comment adavidenko/comment:1.0
docker run -d --network=reddit -p 9292:9292 adavidenko/ui:2.0
```
Перезагрузка контейнера и проверка, что наши данные сохранились - http://51.250.90.59:9292

Зачистка
```
docker rm $(docker ps -q)
docker rmi $(docker images -q)
docker-machine rm docker-host
yc compute instance delete docker-host
```

## Лекция 18

### Docker-4
Создадим Docker хост в Yandex Cloud
```
yc compute instance create
```
Установка docker-machine
```
docker-machine create
```
Смотрим
```
docker-machine ls
```
Переключаемся
```
eval $(docker-machine env docker-hosts)
```
Запустить контейнер с использованием none-драйвера
```
docker run -ti --rm --network none joffotron/docker-net-tools -c ifconfig
```
Запустить контейнер в сетевом пространстве docker-хоста
```
docker run -ti --rm --network host joffotron/docker-net-tools -c ifconfig
```
Создадь bridge-сеть в docker
```
docker network create reddit --driver bridge
```
Запустить наш проект reddit с использованием bridge-сети
```
docker run -d --network=reddit --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db mongo:latest
docker run -d --network=reddit --network-alias=post adavidenko/post:1.0
docker run -d --network=reddit --network-alias=comment adavidenko/comment:1.0
docker run -d --network=reddit -p 9292:9292 adavidenko/ui:2.0
```

Запустить  проект в 2-х bridge сетях.
```
Создать docker-сети
docker network create back_net --subnet=10.0.2.0/24
docker network create front_net --subnet=10.0.1.0/24
```
Запустить контейнеры
```
docker run -d --network=back_net  --network-alias=post_db --network-alias=comment_db -v reddit_db:/data/db --name mongo_db mongo:latest
docker run -d --network=back_net  --network-alias=post --name post adavidenko/post:1.0
docker run -d --network=back_net  --network-alias=comment --name comment adavidenko/comment:1.0
docker run -d --network=front_net -p 9292:9292 --name ui adavidenko/ui:2.0
```
Docker при инициализации контейнера может подключить к нему только 1 сеть
 - По этому необходимо подключить контейнеры ко второй сети
```
docker network connect front_net post
docker network connect front_net comment
```

docker-compose

Выполнить:
```
export USERNAME=adavidenko
docker-compose up -d
docker-compose ps
```
COMPOSE_PROJECT_NAME
```
Задает имя проекта. Это значение добавляется вместе с именем службы в контейнер при запуске. Например, если ваш проект называется myapp и включает в себя две службы db и web, Compose запускает контейнеры с именами myapp-db-1 и myapp-web-1 соответственно.
Установка этого параметра необязательна. Если вы не установите это, COMPOSE_PROJECT_NAME по умолчанию будет базовым именем каталога проекта.
```
## Лекция gitlab-ci
### В процессе сделано:
 - Добавить remote
```
git checkout -b gitlab-ci-1
git remote add gitlab http://51.250.94.138/homework/example.git
```
 - Установить раннер внутри докера
```
docker run -d --name gitlab-runner --restart always -v /srv/gitlabrunner/config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest
```
 - Зарегистрировать раннер
```
docker exec -it gitlab-runner gitlab-runner register
--url http://51.250.94.138/
--non-interactive
--locked=false
--name DockerRunner
--executor docker
--docker-image alpine:latest
--registration-token r4cGznxa45P5wx_B9CHo
--tag-list "linux,xenial,ubuntu,docker"
--run-untagged
```
 - Добавить исходный код reddit в репозиторий:
```
git clone https://github.com/express42/reddit.git && rm -rf ./reddit/.git
git add reddit/
git commit -m "Add reddit app"
git push gitlab gitlab-ci-1
```
### Как запустить проект:
 - docker-compose up -d

### Как проверить работоспособность:
 - http://51.250.94.138

## Лекция monitoring
### Создадим инстанс на YC
```
yc compute instance create \
--name docker-host \
--zone ru-central1-a \
--network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 \
--create-boot-disk image-folder-id=standard-images,image-family=ubuntu-1804-lts,size=15 \
--ssh-key ~/.ssh/appuser.pub
```
### Уточним внешний ip
```
one_to_one_nat:
      address: 51.250.88.60
      ip_version: IPV4
```
### Запустим docker-machine
```
docker-machine create \
--driver generic \
--generic-ip-address=51.250.88.60 \
--generic-ssh-user yc-user \
--generic-ssh-key ~/.ssh/appuser \
docker-host
```
### переключимся на работу с docker-machine
```
eval $(docker-machine env docker-host)
```
### Уточнить ip docker-machine
```
docker-machine ip docker-host
```
### Запустить prometheus
```
docker run --rm -p 9090:9090 -d --name prometheus prom/prometheus
```
### Соберем образ своего prometheus
```
export USER_NAME=adavidenko
docker build -t $USER_NAME/prometheus .
```
### Создадим образы через docker_build.sh
```
for i in ui post-py comment; do cd src/$i; bash docker_build.sh; cd -; done
```
### Создадим описание нашего docker-compose и запустим его
```
docker-compose up -d
```
### Проверим как отзываются метрики если потушим один из контейнеров
```
docker-compose stop post
docker-compose start post
```
### Добавим node_exporter, проверим состояние CPU, дадим нагрузку
```
docker-machine ssh docker-host
yes > /dev/null
```
### Задлогинимся и запушим свои образы на DockerHub
```
docker login
docker push $USER_NAME/ui
docker push $USER_NAME/comment
docker push $USER_NAME/post
docker push $USER_NAME/prometheus
```
### Готовые образы

 - https://hub.docker.com/layers/231788293/adavidenko/ui/latest/images/sha256-deadb01ce2592e64ac49705c7629b25029b230bd719a6f09cd827667edce2018?context=repo
 - https://hub.docker.com/layers/231788736/adavidenko/post/latest/images/sha256-84d51eb6bf140211a282f353d55125af4f32a8ea63a4e15e745ee8f9a5e2a8ad?context=repo
 - https://hub.docker.com/layers/231788552/adavidenko/comment/latest/images/sha256-f5e72b42b0bba6421836d32e375a763b00d4123ee442153c363f82e8ca84af78?context=repo
 - https://hub.docker.com/layers/231788854/adavidenko/prometheus/latest/images/sha256-f7993235c20ecad2037780c71ea81d73c8bd48bfab88e92e2d3873c256a68145?context=repo

### Зачистка
```
docker-compose down
docker rm $(docker ps -q)
docker rmi $(docker images -q)
docker-machine rm docker-host
yc compute instance delete docker-host
```

## Лекция logging
### В процессе сделано:
- Подготовка окружения
- Логирование Docker-контейнеров
- Сбор неструктурированных логов
- Визуализация логов
- Сбор структурированных логов
- Распределенный трейсинг

## Лекция kubernetes-1
### 1. Создать 2 ВМ на Яндекс Облаке (1нода-мастер, 2нода-воркер)
```
Характеристики
RAM 4
CPU 4
SSD 40 GB
```
### 2. Зайти на каждую ВМ и установить
```
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt install docker.io
```
### 3. Зайти на ноду мастер и выполнить
```
sudo kubeadm init --apiserver-cert-extra-sans=51.250.92.72 --apiserver-advertise-address=0.0.0.0 --control-plane-endpoint=51.250.92.72 --pod-network-cidr=10.244.0.0/16
В итоге получим уведомление и дальнейшую инструкцию
```
### 4. Перекинуть kube_config к себе на локальную машину
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/confi
```
### 5. Зайти на ноду воркер и выполнить команду
```
kubeadm join 51.250.92.72:6443 --token yk50x3.u3kwb0njwyhafqec \
        --discovery-token-ca-cert-hash sha256:d44c17e0dd7066d95dc837333dcb027a165acc2054fb0def541925148a9e87ca
```
### 6. На локальной машине выполнить проверку
```
kubectl get nodes
kubectl describe node node-name
```
### 7. Скачать calico и настроить
```
curl https://projectcalico.docs.tigera.io/manifests/calico.yaml -O
Обратите внимание, что нужно изменить параметр в ранее скачанном файле calico.yaml
Значение CALICO_IPV4POOL_CIDR должно быть 10.244.0.0/16
kubectl apply -f calico.yaml
```
### 8. Проверить, что ноды в статусе Ready
```
kubectl get nodes
```
### 9. Можно запускать манифесты
```
kubectl apply -f <filename>
```

## Лекция kubernetes-2
### Minikube-кластер
Запустим наш Minikube-кластер
- minikube start --kubernetes-version 1.19.7

Проверим Minikube-кластер
- kubectl get nodes

Создать cluster
- kubectl config set-cluster ... cluster_name

Создать данные пользователя (credentials)
- kubectl config set-credentials ... user_name

Создать контекст
```
kubectl config set-context context_name \
 --cluster=cluster_name \
 --user=user_name
```
Использовать контекст
- kubectl config use-context context_name

Текущий контекст можно увидеть так:
- kubeclt config current-context

Список всех контекстов можно увидеть так:
- kubectl config get-contexts

Запустим в Minikube ui-компонент
- kubectl apply -f ui-deployment.yml
- kubectl get deployment
- kubectl apply -f ./kubernetes/reddit
- kubectl get pods --selector component=ui
- kubectl port-forward <pod-name> 8080:9292
### Запуск k8s в YC
- yc managed-kubernetes cluster get-credentials otus-claster-k8s --external
- kubectl config current-context
- kubectl apply -f ./kubernetes/reddit/dev-namespace.yml
- kubectl apply -f ./kubernetes/reddit/ -n dev
- kubectl get nodes -o wide
- kubectl describe service ui -n dev | grep NodePort

## Лекция kubernetes-3
### Ознакомились с:
- Ingress Controller
- Ingress
- Secret
- TLS
- LoadBalancer Service
- Network Policies
- PersistentVolumes
- PersistentVolumeClaims

### Установка Ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingressnginx/controller-v0.34.1/deploy/static/provider/cloud/deploy.yaml
kubectl apply -f ui-ingress.yml -n dev
kubectl get ingress
kubectl get ingress -n dev
```
### Создание секрета с сертификатом для ingress
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=35.190.66.90"
kubectl create secret tls ui-ingress --key tls.key --cert tls.crt -n dev
kubectl describe secret ui-ingress -n dev
```
### Network Policy для Google Cloud
```
gcloud beta container clusters list
gcloud beta container clusters update <cluster-name> --zone=us-central1-a --update-addons=NetworkPolicy=ENABLED
gcloud beta container clusters update <cluster-name> --zone=us-central1-a --enable-network-policy
```
### PersitentVolume и PVC
- Для начала создадим диск в ya.cloud командой:
```
yc compute disk create \
 --name k8s \
 --size 4 \
 --description "disk for k8s"
```
- Посмотреть список дисков можно следующей командой:
```
yc compute disk list
``` 
## Лекция kubernetes-4
- Познакомились Helm
- При помощи Helm развернули Gitlab в Kubernetes
- Запустили CI/CD конвейер в Kubernetes
