**Dockerfile**
```Dockerfile
# Используем Ubuntu-based образ вместо Alpine
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

ENTRYPOINT ["./entrypoint.sh"]
```

**entrypoint.sh**
```bash
#!/bin/bash

# Переменные окружения с дефолтами
JVM_OPTS=${JVM_OPTS:-"-Xmx7G -Xms2G"}
JAR_FILE=${JAR_FILE:-"paper-1.21.10-117.jar"}
JAVA_ARGS=${JAVA_ARGS:-"-XX:+UseG1GC -XX:+ParallelRefProcEnabled"}
ENABLE_RCON=${ENABLE_RCON:-"true"}
RCON_PORT=${RCON_PORT:-25575}
RCON_PASSWORD=${RCON_PASSWORD:-"minecraft"}

# Переходим в директорию сервера
cd /opt/minecraft/server

echo "=========================================="
echo "Minecraft Paper Server 1.21.10"
echo "=========================================="
echo "JVM Options: $JVM_OPTS"
echo "JAR File: $JAR_FILE"
echo "Java Args: $JAVA_ARGS"
echo "Working Directory: $(pwd)"
echo "User: $(whoami) (UID: $(id -u), GID: $(id -g))"
echo "=========================================="

# Проверка наличия JAR файла
if [ ! -f "$JAR_FILE" ]; then
    echo "❌ ERROR: JAR file '$JAR_FILE' not found in $(pwd)!"
    echo "Available JAR files:"
    ls -la *.jar 2>/dev/null || echo "   No JAR files found"
    echo "=========================================="
    exit 1
fi

# Настройка EULA
if [ ! -f "eula.txt" ]; then
    echo "ℹ️  eula.txt not found. Creating and accepting EULA..."
    echo "eula=true" > eula.txt
elif ! grep -q "eula=true" eula.txt; then
    echo "ℹ️  EULA not accepted. Setting eula=true..."
    sed -i 's/eula=false/eula=true/g' eula.txt 2>/dev/null || echo "eula=true" > eula.txt
fi

# Настройка RCON (если включён)
if [ "$ENABLE_RCON" = "true" ]; then
    echo "ℹ️  Configuring RCON..."
    if [ ! -f "server.properties" ]; then
        echo "enable-rcon=true" >> server.properties
        echo "rcon.port=$RCON_PORT" >> server.properties
        echo "rcon.password=$RCON_PASSWORD" >> server.properties
    else
        # Обновляем или добавляем RCON настройки
        sed -i '/^enable-rcon=/d' server.properties
        sed -i '/^rcon.port=/d' server.properties
        sed -i '/^rcon.password=/d' server.properties
        echo "enable-rcon=true" >> server.properties
        echo "rcon.port=$RCON_PORT" >> server.properties
        echo "rcon.password=$RCON_PASSWORD" >> server.properties
    fi
fi

# Создание необходимых папок
mkdir -p logs backups world world_nether world_the_end plugins

echo "✅ Server configured successfully"
echo "⏳ Starting Minecraft server..."
echo "=========================================="

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
    container_name: minecraft-paper-12110
    restart: unless-stopped
    network_mode: "host"
    stop_grace_period: 90s  # Увеличим для сохранения больших миров
    # Поддержка graceful shutdown
    stop_signal: SIGTERM

    # Используем новый синтаксис для ресурсов (поддерживается в v2)
    deploy:
      resources:
        limits:
          memory: 8G
          cpus: '2.0'
        reservations:
          memory: 3G
          cpus: '0.5'

    environment:
      # Основные JVM настройки
      JVM_OPTS: "-Xmx7G -Xms2G"
      # Имя JAR файла (обязательно обновите под ваш файл!)
      JAR_FILE: "paper-1.21.10-117.jar"
      # Дополнительные оптимизации для Minecraft в Docker
      JAVA_ARGS: "-XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:+UnlockExperimentalVMOptions -XX:MaxGCPauseMillis=100 -XX:+DisableExplicitGC -Dlog4j2.formatMsgNoLookups=true"
      # Переменные для управления сервером
      ENABLE_RCON: "true"
      RCON_PORT: "25575"
      RCON_PASSWORD: "${RCON_PASSWORD:-minecraft}"

    volumes:
      # Основные данные сервера
      - ./data:/opt/minecraft/server
      # Для бэкапов (опционально)
      - ./backups:/opt/minecraft/backups

    # Оптимизация для производительности
    tmpfs:
      - /tmp:exec,size=512M
      - /opt/minecraft/server/cache:exec,size=256M

    # Настройки здоровья сервера (опционально)
    healthcheck:
      test: ["CMD", "rcon-cli", "list"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 60s

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
  # Можно использовать вместо папки ./data
  # minecraft-data:
  #   name: minecraft-server-data
  backups:
    driver: local
```

```bash
docker-compose build
docker-compose up -d
docker-compose logs -f

docker attach mc-paper-server

docker-compose stop / kill / restart
```

- **Обновить Paper.jar:**
1. Остановите контейнер: `docker-compose stop`.
2. Замените файл `./data/paper-1.21.1-....jar` на новый.
3. Измените имя файла в `docker-compose.yml` (переменная `JAR_FILE`) или переименуйте новый jar под старое имя.
4. Запустите: `docker-compose up -d`.
- **Сделать бэкап:**  
Просто архивируйте папку `./data` (кроме possibly `cache` или временных файлов).

 - Присоединиться
 ```
docker attach minecraft-paper-12110 
 ```
Выход: `Ctrl+P`, `Ctrl+Q` **_(НЕ Ctrl+C - это остановит сервер!)_**

- Отправить команду без входа
```
docker compose exec minecraft rcon-cli "say Hello"
```
_(нужен включенный RCON в server.properties)_

# Список дел

бляяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяяя
чинить всё

узнать про RCON

перенести на 1.21.11 ?