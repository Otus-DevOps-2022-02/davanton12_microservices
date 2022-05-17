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
