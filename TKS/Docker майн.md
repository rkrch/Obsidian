**Dockerfile**
```Dockerfile
# Используем официальный образ OpenJDK с нужной версией.
# Paper 1.21.1+ требует Java 21+. Проверьте актуальную версию для вашего Paper.
FROM eclipse-temurin:21-jdk-jammy
# Создадим пользователя 'minecraft' для безопасности (не запускаем от root).
RUN useradd -ms /bin/bash minecraft && \
    mkdir -p /opt/minecraft/server && \
    chown -R minecraft:minecraft /opt/minecraft
# Переключаемся на пользователя 'minecraft'
USER minecraft
WORKDIR /opt/minecraft/server
# Копируем стартовый скрипт (создадим его ниже)
COPY --chown=minecraft:minecraft entrypoint.sh .
# Делаем скрипт исполняемым
RUN chmod +x entrypoint.sh
# Объявляем том для данных. Это гарантирует, что данные не потеряются.
VOLUME ["/opt/minecraft/server"]
# Порт Minecraft (можно изменить в server.properties, но здесь для информации)
EXPOSE 25565
# Точка входа - наш скрипт
ENTRYPOINT ["./entrypoint.sh"]
```

**entrypoint.sh**
```bash
#!/bin/bash
# Переменные окружения с дефолтными значениями.
# Их можно переопределить через docker-compose.yml или Docker.
JVM_OPTS=${JVM_OPTS:-"-Xms4G -Xmx4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled"}
JAR_FILE=${JAR_FILE:-"paper-1.21.1-1234.jar"} # ЗАМЕНИТЕ на ваш точный jar-файл!
echo "Starting Minecraft server with JVM options: $JVM_OPTS"
echo "Using JAR file: $JAR_FILE"
# Запуск сервера. Nogui обязательно, так как в контейнере нет GUI.
exec java $JVM_OPTS -jar $JAR_FILE nogui
```

```bash
chmod +x ~/minecraft-docker/entrypoint.sh
```

**docker-compose.yml**
```yaml
version: '3.8'
services:
  minecraft:
    build: .  # Собираем образ из Dockerfile в текущей директории
    container_name: mc-paper-server
    restart: unless-stopped  # Автоматический перезапустрок при падении или рестарте системы
    environment:
      # Переопределяем JVM флаги. Настройте под ваше железо!
      JVM_OPTS: "-Xms4G -Xmx4G -XX:+UseG1GC -XX:+ParallelRefProcEnabled -XX:+UnlockExperimentalVMOptions -XX:MaxGCPauseMillis=100"
      JAR_FILE: "paper-1.21.1-1234.jar" # Убедитесь, что имя файла совпадает
    ports:
      - "25565:25565"  # Основной порт. Левая часть - хост, правая - контейнер.
    volumes:
      # Монтируем локальную папку 'data' внутрь контейнера.
      # ВСЕ данные сервера будут храниться на хосте и доступны для бэкапов.
      - ./data:/opt/minecraft/server
    # Ограничения ресурсов (опционально, но рекомендуется)
    deploy:
      resources:
        limits:
          memory: 5G  # Чуть больше, чем Xmx
        reservations:
          memory: 1G
    # Настройки для корректной работы Minecraft
    stdin_open: true  # Позволяет отправлять команды через 'docker attach'
    tty: true
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