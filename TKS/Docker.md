[[Docker майн]]
Из докер файла собирается образ, на основе образа запускается контейнер.

Dockerfile - набор инструкций,
Image - уже готовое к запуску приложение
Container - экземпляр приложения

```Dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get -y instll nginix
EXPOSE 80/tcp
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
```
 
```bash
 docker build -t myapp./ # -t - возможность удаленного взаимодействия
 docker run -d -p 80:80 myapp
 
 
 docker run --name name -d -p 80:80 myapp
 docker run --rm удалит сразу после выполнения
  # Другие флаги: docker run --help
 docker start name # запускает в фоне , вместо name можно ID
 docker start -i name # -i - не в фоне, весь вывод показывает
 docker stop name 
 docker kill name # в крайнем случае
 
 docker rm name # удаляет контейнер
 docker container prune # удаляет все остановленные контейнеры
 docker rmi imgname:latest# удаляет образ (Image)
  # Сначала удаляется контейнер! Только потом можно удалить образ
 
 docker ps -a # выводит существующие контейнеры, в тч остановленные
 docker ps # работающие  
 # -q - только ID
 docker images
 
 # !
 docker stats # показывает потребляемые контейнером ресурсы
 
 docker exec name `command` # выполнит команду в указанном контейнере
 docker build . -t imgname:tag # `.` - путь
 
 
 # Коды завершения
 # 137 - принудительное
 # 0 - закончило работу и само закрылось.
```

```bash
#с репо
docker pull python:latest # качать моэно сразу в `run`
#если тэг не писать, то по умолчанию latest

docker commit name name2:tag
# создаёт из изменённого относительно образа контейнера новый образ 

docker builder prune # (-a) - удаляет кэш
ducker system prune # удалит остановленные контейнеры, неиспользуемые сети, образы, кэш
# -a - удалит ещё и изображения и всё вообще кроме работающего

```
## Docker compose

мяу
```

```
## Kubernetes

мяу
```

```



## Linux (перенести)

```
rm -r 
```