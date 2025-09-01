---
tags:
  - inbox
---
Создание контейнера docker run container_name
Просмотр контейнеров docker ps 
просмотр всех контейнеров docker ps -a
Удаление контейнера docker rm hash
Просмотр имеджей docker images
Удаление образа docker rmi 
Загрузить образ docker pull 
Запуск контейнера docker start 
Пауза docker pause
Снять с паузы docker unpause 
Запуск в фоне  docker run -d 
Завершить контейнер docker stop 
Убить процесс docker kill 

-rm удалить после завершения 
dokcer inspect 
docker status 
dokcer logs -f
docker exec -it <container-name-or-id> bash
docker save alpine -o alpine-image.tar
docker load -i alpine-image.tar
docker export -o alpine-fs.tar 8f4ae688ed8a
docker import alpine-fs.tar alpine-fs