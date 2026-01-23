```
ssh server@192.168.1.35
http://172.18.55.199:8100
ssh server@172.18.55.199
```
http://172.18.55.199/
## Сейчас:

**Dockerfile**
```Dockerfile
# Debian-based образ
FROM eclipse-temurin:21-jdk-jammy

# Build arguments
ARG USER_ID=1001
ARG GROUP_ID=1001

# Создаем пользователя в Ubuntu/Debian
RUN groupadd -g ${GROUP_ID} minecraft && \
    useradd -m -u ${USER_ID} -g minecraft -s /bin/bash minecraft && \
    mkdir -p /opt/minecraft/server && \
    chown -R minecraft:minecraft /opt/minecraft

USER minecraft
WORKDIR /opt/minecraft

COPY entrypoint.sh .

VOLUME ["/opt/minecraft/server"]

EXPOSE 25565

EXPOSE 8100

EXPOSE 24454

ENTRYPOINT ["./entrypoint.sh"]
```

**entrypoint.sh**
```bash
#!/bin/bash

# Переменные окружения с дефолтами
JVM_OPTS=${JVM_OPTS:-"-Xmx3G -Xms2G"}
JAR_FILE=${JAR_FILE:-"paper-1.21.10-117.jar"}
JAVA_ARGS=${JAVA_ARGS:-"-XX:+UseG1GC -XX:G1HeapRegionSize=8M -XX:MaxGCPauseMillis=200"}
ENABLE_RCON=${ENABLE_RCON:-"false"}

# Переходим в директорию сервера
cd /opt/minecraft/server

echo "JVM Options: $JVM_OPTS"
echo "JAR File: $JAR_FILE"
echo "Java Args: $JAVA_ARGS"
echo "Working Directory: $(pwd)"
echo "User: $(whoami) (UID: $(id -u), GID: $(id -g))"
echo "=========================================="

# Проверка наличия JAR файла
if [ ! -f "$JAR_FILE" ]; then
    echo "JAR file '$JAR_FILE' not found in $(pwd)!"
    echo "Available JAR files:"
    ls -la *.jar 2>/dev/null || echo "   No JAR files found"
    echo "=========================================="
    exit 1
fi

# Настройка EULA
if [ ! -f "eula.txt" ]; then
    echo "eula.txt not found. Creating and accepting EULA..."
    echo "eula=true" > eula.txt
elif ! grep -q "eula=true" eula.txt; then
    echo "EULA not accepted. Setting eula=true..."
    sed -i 's/eula=false/eula=true/g' eula.txt 2>/dev/null || echo "eula=true" > eula.txt
fi

# Создание необходимых папок
mkdir -p logs backups world world_nether world_the_end plugins

echo " Server configured successfully"
echo " Starting Minecraft server..."

# Запуск сервера
exec java $JVM_OPTS $JAVA_ARGS -jar "$JAR_FILE" nogui
```

```bash
chmod +x ~/minecraft-docker/entrypoint.sh
```

**docker-compose.yml**
```yaml
services:
  minecraft:
    build:
      context: .
      args:
        USER_ID: ${USER_ID:-1001}
        GROUP_ID: ${GROUP_ID:-1001}
    container_name: mine-paper
    restart: unless-stopped
    network_mode: "host"
    stop_grace_period: 90s  # Увеличим для сохранения больших миров
    # Поддержка graceful shutdown
    stop_signal: SIGTERM

    # Используем новый синтаксис для ресурсов (поддерживается в v2)
    deploy:
      resources:
        limits:
          memory: 3G
          cpus: '2.0'
        reservations:
          memory: 3G
          cpus: '0.5'

    environment:
      # Основные JVM настройки
      JVM_OPTS: "-Xmx2G -Xms3G"
      # Имя JAR файла (обязательно обновите под ваш файл!)
      JAR_FILE: "paper-1.21.10-117.jar" # paper-1.21.11-8.jar
      # Дополнительные оптимизации для Minecraft в Docker
      JAVA_ARGS: "-XX:+UseG1GC -XX:G1HeapRegionSize=8M -XX:MaxGCPauseMillis=200"

    volumes:
      # Основные данные сервера
      - ./data:/opt/minecraft/server
      # Для бэкапов (опционально)
      - ./backups:/opt/minecraft/backups

    # Логирование
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

    # Настройки для консоли
    stdin_open: true  # docker attach
    tty: true

 #Тома для постоянного хранения (альтернатива папкам)
volumes:
  backups:
    driver: local
```

```bash
docker-compose build
docker-compose up -d
docker-compose logs -f


docker-compose stop / kill / restart

docker attach ...
```
 **_Выход: `Ctrl+P`, `Ctrl+Q` (Ctrl+C  остановит сервер!!!)_**
 
- **Обновить Paper.jar:**
1. Остановите контейнер: `docker-compose stop`.
2. Замените файл `./data/paper-1.21.1-....jar` на новый.
3. Изменить имя файла в `docker-compose.yml` (переменная `JAR_FILE`)
4.  `docker-compose up -d`.

## Test
**docker**

## Список дел

prometheus 
grafana




docker exec -it atm10-skyblock bash
cd /opt/minecraft
ls -la
echo "eula=true" > eula.txt









 1535  docker-compose -f docker-compose.web.yml restart
 1536  ls
 1537  ll
 1538  cat configs/nginx.conf
 1539  curl -I http://localhost/css/style.css
 1540  mv configs/nginx.conf configs/nginx.conf.backup
 1541  docker-compose -f docker-compose.web.yml restart
 1542  docker exec nginx-proxy nginx -t
 1543  docker exec nginx-proxy ls -la /etc/nginx/conf.d/
 1544  curl http://localhost/ | head -20
 1545  docker-compose -f docker-compose.web.yml restart
 1546  docker stats
 1547  nc
 1548  nc 172.18.55.199 25565
 1549  cd .
 1550  cd ..
 1551  cd mine
 1552  ls
 1553  nano entrypoint.sh
 1554  nano docker-compose.yml
 1555  docker-compose -f docker-compose.yml down
 1556  cd ..
 1557  cd web
 1558  nc 172.18.55.199 25565
 1559  nc -z 172.18.55.199 25565
 1560  cd ..
 1561  cd mine
 1562  docker-compose -f docker-compose.yml up -d
 1563  cd ..
 1564  cd web
 1565  docker-compose -f docker-compose.web.yml restart
 1566  docker stats
 1567  ps
 1568  htop
 1569  jobs
 1570  ps aux
 1571  uptime
 1572  free
 1573  free -h
 1574  nice
 1575  man nice
 1576  ps al
 1577  ps alx
 1578  ps al
 1579  ps al -р
 1580  ps al -h
 1581  history
 1582  docker-compose.web.yml restart
 1583  cd modded/
 1584  docker-compose.web.yml restart
 1585  docker-compose -f docker-compose.mod.yml logs -f
 1586  docker exec -it atm10-skyblock bash
 1587  docker-compose -f docker-compose.mod.yml down
 1588  docker exec -it atm10-skyblock bash
 1589  ды
 1590  ll
 1591  nano server.properties
 1592  cat Dockerfile
 1593  cat docker-compose.mod.yml
 1594  nano docker-compose.mod.yml
 1595  history



# Перезапуск
docker-compose -f docker-compose.mod.yml restart

# Остановка
docker-compose -f docker-compose.mod.yml down

# Запуск
docker-compose -f docker-compose.mod.yml up -d

# Просмотр логов
docker-compose -f docker-compose.mod.yml logs -f

# Проверка статуса
docker-compose -f docker-compose.mod.yml ps




serverver
25575
rcon -p 25575 -P serverver -a localhost "/atl force_refresh"

mcrcon-nsg -H localhost -P 25575 -p serverver -t