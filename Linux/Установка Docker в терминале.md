## **Установка Docker на Ubuntu**

### 1. **Обновление пакетного менеджера**
```bash
sudo apt update && sudo apt upgrade -y
```
- `sudo` - выполнение с правами администратора
- `apt update` - обновление списка доступных пакетов
- `apt upgrade -y` - установка обновлений (`-y` = автоматическое "yes" на все вопросы)

### 2. **Установка зависимостей**
```bash
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```
**Объяснение пакетов:**
- `apt-transport-https` - позволяет apt работать с HTTPS-репозиториями
- `ca-certificates` - сертификаты для проверки SSL/TLS
- `curl` - утилита для загрузки файлов из интернета
- `gnupg` - инструмент для работы с криптографическими ключами
- `lsb-release` - предоставляет информацию о версии Ubuntu

### 3. **Добавление GPG-ключа Docker**
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
**Разбор команды:**
- `curl -fsSL`:
  - `-f` - fail silently на ошибках HTTP
  - `-s` - silent mode (без прогресс-бара)
  - `-S` - показывать ошибки даже в silent mode
  - `-L` - следовать перенаправлениям
- `gpg --dearmor` - конвертирует ключ из ASCII в бинарный формат
- `-o /usr/share/keyrings/...` - сохраняет ключ в системную директорию

### 4. **Добавление репозитория Docker**
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
**Что происходит:**
- `[arch=amd64]` - указываем архитектуру процессора
- `signed-by=...` - используем добавленный ключ для проверки пакетов
- `$(lsb_release -cs)` - автоматически подставляет кодовое имя твоей Ubuntu (focal)
- `stable` - используем стабильную версию Docker
- `tee /etc/apt/sources.list.d/docker.list` - создает файл в директории источников apt

### 5. **Установка Docker**
```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io
```
**Устанавливаемые компоненты:**
- `docker-ce` - Community Edition (основной пакет)
- `docker-ce-cli` - Command Line Interface
- `containerd.io` - низкоуровневый демон для управления контейнерами

### 6. **Настройка прав пользователя**
```bash
sudo usermod -aG docker $USER
```
- `usermod` - изменяет параметры пользователя
- `-aG docker` - добавляет (append) пользователя в группу (Group) "docker"
- `$USER` - автоматически подставляет текущего пользователя

### 7. **Перезагрузка**
```bash
sudo reboot
```
**Зачем:** Изменения групп вступают в силу только после перелогина

##  **Проверка установки**

После перезагрузки:
```bash
# Проверяем версию Docker
docker --version

# Проверяем статус демона Docker
sudo systemctl status docker

# Запускаем тестовый контейнер (должен работать без sudo!)
docker run hello-world
```

##  **Альтернативный способ установки**

Если хочешь более простой путь:
```bash
# Автоматический скрипт установки
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## ⚠️ **Важные моменты:**

1. **Репозиторий Docker** гарантирует свежие версии (в отличие от `docker.io` из репозиториев Ubuntu)
2. **GPG-ключ** обеспечивает безопасность - пакеты проверяются на подлинность
3. **Группа docker** избавляет от необходимости постоянно писать `sudo`
4. **Containerd** - это "мозг" Docker, который реально управляет контейнерами


nginx-proxy
 docker-compose -f docker-compose.web.yml up -d
 docker-compose -f ~/web/docker-compose.web.yml restart