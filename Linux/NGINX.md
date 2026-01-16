```
docker-compose -f docker-compose.web.yml restart
```



##  Структура:
```
 ~/web/
 ├── docker-compose.web.yml
 ├── Dockerfile
 ├── logs/
 │   └─
 ├── ssl/
 │   └─
 ├── sites/
 │   └── main.conf
 └── www/
     ├── css/
     │   └── style.css      <-- новый файл
	 ├── images/
	 │   └─
     ├── js/
     │   └── app.js         <-- новый файл
     ├── icons/              <-- для иконок и логотипов
     └── index.html          <-- новый главный файл
```


## ~/web/sites$ cat main.conf
server {
    listen 80;
    server_name _;

    # Главная страница
    # Корень сайта - статика
    location / {
        root /var/www/html;
        index index.html;
        try_files $uri $uri/ =404;
    }

    # CSS, JS, изображения
    location ~* \.(css|js|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        root /var/www/html;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    location /grafana/ {
        # Просто редирект на прямой порт
        return 302 http://172.18.55.199:3000;
    }

    # Редирект без слэша
    location = /grafana {
        return 302 /grafana/;
    }

    # API для онлайна (заглушка)
    location /api/online {
        default_type application/json;
        return 200 '{"online": 3, "max": 20}';
    }
}





## ~/nginx-docker/configs/nginx.conf
```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Формат логов
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" "$http_user_agent"';

    access_log /var/log/nginx/access.log main;

    # Оптимизации
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 10M;

    # Защита
    server_tokens off;

    # Загружаем конфиги сайтов
    includ
```


## `~/web/www/js/app.js` 
```js
// app.js - Интерактивные функции для сайта

// Копирование IP адреса сервера
function copyServerIP() {
    const ip = '172.18.55.199:25565';
    
    // Используем современный API, если доступен
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(ip).then(() => {
            showNotification('✅ IP адрес скопирован в буфер обмена!');
        }).catch(err => {
            console.error('Ошибка при копировании:', err);
            fallbackCopy(ip);
        });
    } else {
        // Старый метод для поддержки всех браузеров
        fallbackCopy(ip);
    }
}

// Резервный метод копирования
function fallbackCopy(text) {
    const textArea = document.createElement('textarea');
    textArea.value = text;
    textArea.style.position = 'fixed';
    textArea.style.opacity = '0';
    document.body.appendChild(textArea);
    textArea.focus();
    textArea.select();
    
    try {
        const successful = document.execCommand('copy');
        if (successful) {
            showNotification('✅ IP адрес скопирован в буфер обмена!');
        } else {
            showNotification('❌ Не удалось скопировать. Скопируйте вручную: ' + text);
        }
    } catch (err) {
        console.error('Ошибка при резервном копировании:', err);
        showNotification('❌ Ошибка копирования. Скопируйте вручную: ' + text);
    }
    
    document.body.removeChild(textArea);
}

// Показ уведомления
function showNotification(message) {
    // Удаляем предыдущее уведомление, если есть
    const oldNotification = document.querySelector('.copy-notification');
    if (oldNotification) {
        oldNotification.remove();
    }
    
    // Создаем новое уведомление
    const notification = document.createElement('div');
    notification.className = 'copy-notification';
    notification.textContent = message;
    notification.style.cssText = `
        position: fixed;
        top: 20px;
        right: 20px;
        background: rgba(0, 30, 0, 0.9);
        color: #00ff00;
        padding: 15px 20px;
        border-radius: 4px;
        border: 1px solid #00cc00;
        font-family: 'JetBrains Mono', monospace;
        z-index: 1000;
        box-shadow: 0 5px 15px rgba(0, 0, 0, 0.5);
        animation: slideIn 0.3s ease, fadeOut 0.3s ease 2.7s;
        max-width: 300px;
    `;
    
    document.body.appendChild(notification);
    
    // Удаляем уведомление через 3 секунды
    setTimeout(() => {
        if (notification.parentNode) {
            notification.remove();
        }
    }, 3000);
}

// Добавляем стили для анимации уведомления
const style = document.createElement('style');
style.textContent = `
    @keyframes slideIn {
        from {
            transform: translateX(100%);
            opacity: 0;
        }
        to {
            transform: translateX(0);
            opacity: 1;
        }
    }
    
    @keyframes fadeOut {
        from {
            opacity: 1;
        }
        to {
            opacity: 0;
        }
    }
`;
document.head.appendChild(style);

// Обновление статуса сервера (заглушка для будущей реализации)
async function updateServerStatus() {
    try {
        // Здесь в будущем можно добавить запрос к API сервера
        const response = { online: true, players: 3, maxPlayers: 20 };
        
        // Обновляем индикатор статуса
        const statusIndicator = document.querySelector('.status-indicator');
        if (response.online) {
            statusIndicator.innerHTML = '<span class="pulse"></span> ONLINE (' + response.players + '/' + response.maxPlayers + ' игроков)';
            statusIndicator.className = 'status-indicator online';
        } else {
            statusIndicator.innerHTML = '<span class="pulse" style="background:#ff3333"></span> OFFLINE';
            statusIndicator.className = 'status-indicator offline';
        }
    } catch (error) {
        console.log('Не удалось обновить статус сервера:', error);
    }
}

// Инициализация при загрузке страницы
document.addEventListener('DOMContentLoaded', function() {
    console.log('Сайт мониторинга Minecraft сервера загружен');
    
    // Обновляем статус сервера сразу и каждые 30 секунд
    updateServerStatus();
    setInterval(updateServerStatus, 30000);
    
    // Добавляем обработчик для кнопки копирования
    const copyButton = document.querySelector('.copy-btn');
    if (copyButton) {
        copyButton.addEventListener('click', function(e) {
            e.stopPropagation(); // Чтобы не срабатывал клик по карточке
            copyServerIP();
        });
    }
    
    // Добавляем обработчик для всей карточки с сервером
    const serverCard = document.querySelector('.module-card:nth-child(2)');
    if (serverCard) {
        serverCard.addEventListener('click', function(e) {
            if (!e.target.closest('.copy-btn')) {
                copyServerIP();
            }
        });
    }
    
    // Добавляем текущую дату в футер
    const currentYear = new Date().getFullYear();
    const yearElement = document.querySelector('.footer-copyright p:first-child');
    if (yearElement && currentYear !== 2026) {
        yearElement.textContent = yearElement.textContent.replace('2026', currentYear);
    }
});
```

## `~/web/www/css/style.css`
```css
/* ===== ОСНОВНЫЕ СТИЛИ В СТИЛЕ XATA436.RU ===== */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

:root {
    --bg-primary: #0a0a0a;
    --bg-secondary: #111111;
    --terminal-green: #00ff00;
    --terminal-green-dim: #00cc00;
    --terminal-cyan: #00ffff;
    --text-primary: #f0f0f0;
    --text-secondary: #aaaaaa;
    --text-comment: #666666;
    --border-color: #333333;
    --accent-blue: #0066cc;
    --accent-purple: #9900ff;
    --online: #00ff00;
    --offline: #ff3333;
    --warning: #ffaa00;
}

body {
    font-family: 'JetBrains Mono', 'Ubuntu Mono', 'Courier New', monospace;
    background-color: var(--bg-primary);
    color: var(--text-primary);
    line-height: 1.6;
    font-size: 16px;
    min-height: 100vh;
    padding: 20px;
    background-image: 
        radial-gradient(circle at 10% 20%, rgba(0, 80, 0, 0.05) 0%, transparent 20%),
        radial-gradient(circle at 90% 80%, rgba(0, 100, 255, 0.03) 0%, transparent 20%);
}

.container {
    max-width: 1200px;
    margin: 0 auto;
}

/* ===== ШАПКА В СТИЛЕ ТЕРМИНАЛА ===== */
.terminal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 0 25px;
    border-bottom: 1px solid var(--border-color);
    margin-bottom: 30px;
    flex-wrap: wrap;
}

.hostname {
    font-size: 1.8rem;
    font-weight: 700;
}

.hostname .user {
    color: var(--terminal-cyan);
}

.hostname .at {
    color: var(--text-primary);
}

.hostname .host {
    color: var(--terminal-green);
}

.hostname .tilde {
    color: var(--accent-purple);
}

.header-info {
    display: flex;
    gap: 20px;
    align-items: center;
    flex-wrap: wrap;
}

.status-indicator {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 15px;
    border-radius: 4px;
    font-weight: 500;
    font-size: 0.9rem;
}

.status-indicator.online {
    background: rgba(0, 255, 0, 0.1);
    color: var(--online);
    border: 1px solid rgba(0, 255, 0, 0.3);
}

.pulse {
    display: inline-block;
    width: 10px;
    height: 10px;
    background-color: var(--online);
    border-radius: 50%;
    animation: pulse 2s infinite;
}

@keyframes pulse {
    0% { opacity: 1; }
    50% { opacity: 0.4; }
    100% { opacity: 1; }
}

.header-ip {
    color: var(--text-secondary);
    font-size: 0.9rem;
}

.header-ip i {
    margin-right: 5px;
    color: var(--terminal-cyan);
}

/* ===== ОСНОВНОЙ КОНТЕНТ ===== */
.main {
    margin-bottom: 50px;
}

.welcome-section {
    margin-bottom: 40px;
}

.terminal-prompt {
    color: var(--terminal-green);
    font-size: 1.8rem;
    margin-bottom: 15px;
    font-weight: 700;
}

.terminal-prompt .cmd {
    color: var(--text-primary);
}

.comment {
    color: var(--text-comment);
    font-style: italic;
    margin-bottom: 8px;
    font-size: 1.1rem;
}

.section {
    margin-bottom: 50px;
}

.section-title {
    font-size: 1.5rem;
    margin-bottom: 25px;
    color: var(--text-primary);
    display: flex;
    align-items: center;
    gap: 10px;
}

.section-title i {
    color: var(--terminal-green);
}

/* ===== СЕТКА СЕРВИСОВ (MODULES) ===== */
.modules-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 25px;
    margin-bottom: 20px;
}

.module-card {
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
    padding: 25px;
    transition: all 0.3s ease;
    text-decoration: none;
    color: inherit;
    position: relative;
}

.module-card:hover {
    border-color: var(--terminal-green);
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.3);
}

.module-card a {
    text-decoration: none;
    color: inherit;
}

.module-icon {
    font-size: 2.5rem;
    color: var(--terminal-green);
    margin-bottom: 15px;
}

.module-card h3 {
    font-size: 1.3rem;
    margin-bottom: 12px;
    color: var(--text-primary);
}

.module-card p {
    color: var(--text-secondary);
    font-size: 0.95rem;
    line-height: 1.5;
    margin-bottom: 20px;
}

.module-status {
    display: inline-block;
    padding: 5px 12px;
    border-radius: 4px;
    font-size: 0.85rem;
    font-weight: 500;
}

.module-status.online {
    background: rgba(0, 255, 0, 0.1);
    color: var(--online);
    border: 1px solid rgba(0, 255, 0, 0.3);
}

.module-status.restricted {
    background: rgba(255, 170, 0, 0.1);
    color: var(--warning);
    border: 1px solid rgba(255, 170, 0, 0.3);
}

.server-ip {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: rgba(0, 0, 0, 0.3);
    padding: 10px 15px;
    border-radius: 4px;
    margin-top: 15px;
    border: 1px solid var(--border-color);
}

.server-ip code {
    font-family: inherit;
    color: var(--terminal-green);
    font-size: 1rem;
}

.copy-btn {
    background: transparent;
    border: 1px solid var(--terminal-green);
    color: var(--terminal-green);
    width: 36px;
    height: 36px;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
}

.copy-btn:hover {
    background: rgba(0, 255, 0, 0.1);
}

/* ===== СЕТКА ТАРИФОВ / МОНИТОРИНГА ===== */
.pricing-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
    gap: 30px;
    margin-top: 20px;
}

.pricing-card {
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
    padding: 0;
    position: relative;
    transition: all 0.3s ease;
}

.pricing-card:hover {
    border-color: var(--terminal-green);
}

.pricing-card.highlight {
    border-color: var(--accent-purple);
    transform: scale(1.02);
}

.pricing-badge {
    position: absolute;
    top: -12px;
    right: 20px;
    background: var(--accent-purple);
    color: white;
    padding: 5px 15px;
    border-radius: 4px;
    font-size: 0.8rem;
    font-weight: 700;
}

.pricing-header {
    padding: 25px;
    border-bottom: 1px solid var(--border-color);
}

.pricing-header h3 {
    font-size: 1.5rem;
    margin-bottom: 10px;
    color: var(--text-primary);
}

.price {
    font-size: 2.5rem;
    font-weight: 700;
    color: var(--terminal-green);
}

.period {
    font-size: 1rem;
    color: var(--text-secondary);
    font-weight: normal;
}

.features {
    padding: 25px;
    list-style: none;
}

.features li {
    margin-bottom: 15px;
    display: flex;
    align-items: flex-start;
}

.features li i {
    margin-right: 10px;
    margin-top: 3px;
    flex-shrink: 0;
}

.features .green {
    color: var(--terminal-green);
}

.features .red {
    color: #ff3333;
}

.pricing-footer {
    padding: 20px 25px;
    background: rgba(0, 0, 0, 0.2);
    border-top: 1px solid var(--border-color);
    color: var(--text-secondary);
    font-size: 0.9rem;
}

/* ===== БЛОК НОВОСТЕЙ (CHANGELOG) ===== */
.changelog {
    background: var(--bg-secondary);
    border: 1px solid var(--border-color);
    padding: 0;
}

.log-entry {
    padding: 25px;
    border-bottom: 1px solid var(--border-color);
    transition: background 0.2s;
}

.log-entry:last-child {
    border-bottom: none;
}

.log-entry:hover {
    background: rgba(255, 255, 255, 0.02);
}

.log-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
    flex-wrap: wrap;
    gap: 10px;
}

.log-date {
    color: var(--terminal-cyan);
    font-weight: 500;
}

.log-type {
    font-weight: 700;
    font-size: 0.9rem;
    padding: 3px 10px;
    border-radius: 4px;
}

.log-type.info {
    color: var(--terminal-cyan);
    background: rgba(0, 255, 255, 0.1);
    border: 1px solid rgba(0, 255, 255, 0.2);
}

.log-type.update {
    color: var(--terminal-green);
    background: rgba(0, 255, 0, 0.1);
    border: 1px solid rgba(0, 255, 0, 0.2);
}

.log-type.warn {
    color: var(--warning);
    background: rgba(255, 170, 0, 0.1);
    border: 1px solid rgba(255, 170, 0, 0.2);
}

.log-type.feature {
    color: var(--accent-purple);
    background: rgba(153, 0, 255, 0.1);
    border: 1px solid rgba(153, 0, 255, 0.2);
}

.log-title {
    font-size: 1.2rem;
    margin-bottom: 10px;
    color: var(--text-primary);
}

.log-desc {
    color: var(--text-secondary);
    line-height: 1.5;
}

/* ===== ФУТЕР ===== */
.terminal-footer {
    margin-top: 50px;
    padding-top: 30px;
    border-top: 1px solid var(--border-color);
}

.footer-line {
    height: 1px;
    background: linear-gradient(90deg, var(--terminal-green), var(--accent-purple), var(--terminal-cyan));
    margin-bottom: 30px;
}

.footer-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 40px;
    margin-bottom: 30px;
}

.footer-section h4 {
    font-size: 1.1rem;
    margin-bottom: 15px;
    color: var(--text-primary);
    display: flex;
    align-items: center;
    gap: 10px;
}

.tech-tags {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 10px;
}

.tech-tag {
    background: rgba(0, 100, 255, 0.1);
    color: var(--terminal-cyan);
    padding: 5px 12px;
    border-radius: 4px;
    font-size: 0.85rem;
    border: 1px solid rgba(0, 100, 255, 0.2);
}

.footer-section p {
    color: var(--text-secondary);
    margin-bottom: 10px;
    font-size: 0.95rem;
}

.footer-section code {
    background: rgba(0, 0, 0, 0.3);
    padding: 3px 8px;
    border-radius: 4px;
    border: 1px solid var(--border-color);
    color: var(--terminal-green);
    font-family: inherit;
}

.footer-copyright {
    text-align: center;
    color: var(--text-comment);
    font-size: 0.9rem;
    padding-top: 20px;
    border-top: 1px solid var(--border-color);
}

.footer-copyright p {
    margin-bottom: 8px;
}

/* ===== АДАПТИВНОСТЬ ===== */
@media (max-width: 768px) {
    body {
        padding: 15px;
        font-size: 15px;
    }
    
    .terminal-header {
        flex-direction: column;
        align-items: flex-start;
        gap: 15px;
    }
    
    .modules-grid, .pricing-grid {
        grid-template-columns: 1fr;
    }
    
    .terminal-prompt {
        font-size: 1.4rem;
    }
    
    .section-title {
        font-size: 1.3rem;
    }
    
    .footer-content {
        grid-template-columns: 1fr;
        gap: 30px;
    }
}

/* ===== ДОПОЛНИТЕЛЬНЫЕ ЭФФЕКТЫ ===== */
::selection {
    background-color: rgba(0, 255, 0, 0.3);
    color: white;
}

::-webkit-scrollbar {
    width: 10px;
}

::-webkit-scrollbar-track {
    background: var(--bg-primary);
}

::-webkit-scrollbar-thumb {
    background: var(--border-color);
    border-radius: 5px;
}

::-webkit-scrollbar-thumb:hover {
    background: var(--terminal-green-dim);
}
```

## `~/web/www/index.html`
```HTML
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>root@server:~# Minecraft & Мониторинг</title>
    <link rel="stylesheet" href="/css/style.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&family=Ubuntu+Mono&display=swap" rel="stylesheet">
</head>
<body>
    <div class="container">
        <!-- Шапка как в терминале -->
        <header class="terminal-header">
            <div class="hostname">
                <span class="user">server</span><span class="at">@</span><span class="host">minecraft</span><span class="tilde">:~#</span>
            </div>
            <div class="header-info">
                <span class="status-indicator online">
                    <span class="pulse"></span> ONLINE
                </span>
                <span class="header-ip"><i class="fas fa-network-wired"></i> 172.18.55.199</span>
            </div>
        </header>

        <main class="main">
            <!-- Приветственный блок -->
            <section class="welcome-section">
                <h1 class="terminal-prompt"># <span class="cmd">root@server:~# Сервисы, которые так нужны вам</span></h1>
                <p class="comment">// Первая частная разработка кафедры ТКС МИЭТ.</p>
                <p class="comment">// Предоставляем мощные инструменты для игры, учёбы и развлечений.</p>
            </section>

            <!-- Основные сервисы (Modules) -->
            <section class="section">
                <h2 class="section-title"><i class="fas fa-cube"></i> Сервисы (Modules)</h2>
                <div class="modules-grid">
                    <a href="/grafana/" class="module-card">
                        <div class="module-icon"><i class="fas fa-chart-line"></i></div>
                        <h3>Grafana</h3>
                        <p>Панель мониторинга сервера в реальном времени. Графики, метрики, алерты.</p>
                        <span class="module-status online">● ONLINE</span>
                    </a>
                    <div class="module-card" onclick="copyServerIP()">
                        <div class="module-icon"><i class="fas fa-server"></i></div>
                        <h3>Minecraft Server</h3>
                        <p>Основной игровой сервер. Paper 1.21. С плагинами, приватами, экономикой.</p>
                        <div class="server-ip">
                            <code>172.18.55.199:25565</code>
                            <button class="copy-btn"><i class="far fa-copy"></i></button>
                        </div>
                    </div>
                    <a href="http://172.18.55.199:9090" target="_blank" class="module-card">
                        <div class="module-icon"><i class="fas fa-database"></i></div>
                        <h3>Prometheus</h3>
                        <p>Система сбора и хранения метрик. Глубокий мониторинг всех компонентов.</p>
                        <span class="module-status online">● ONLINE</span>
                    </a>
                    <div class="module-card">
                        <div class="module-icon"><i class="fas fa-terminal"></i></div>
                        <h3>SSH / Консоль</h3>
                        <p>Прямой доступ к управлению сервером. Только для администраторов.</p>
                        <span class="module-status restricted"><i class="fas fa-lock"></i> RESTRICTED</span>
                    </div>
                </div>
            </section>

            <!-- Мониторинг (Pricing по аналогии) -->
            <section class="section">
                <h2 class="section-title"><i class="fas fa-desktop"></i> Мониторинг в реальном времени</h2>
                <div class="pricing-grid">
                    <div class="pricing-card">
                        <div class="pricing-header">
                            <h3>БАЗОВЫЙ</h3>
                            <div class="price">0₽ <span class="period">/всегда</span></div>
                        </div>
                        <ul class="features">
                            <li><i class="fas fa-check green"></i> Доступ к игровому серверу 24/7</li>
                            <li><i class="fas fa-check green"></i> Базовая статистика онлайн</li>
                            <li><i class="fas fa-check green"></i> Чат Discord сообщества</li>
                            <li><i class="fas fa-times red"></i> Расширенная аналитика Grafana</li>
                            <li><i class="fas fa-times red"></i> Доступ к сырым метрикам</li>
                        </ul>
                        <div class="pricing-footer">Текущий статус для всех игроков</div>
                    </div>
                    <div class="pricing-card highlight">
                        <div class="pricing-badge">ПОПУЛЯРНЫЙ</div>
                        <div class="pricing-header">
                            <h3>АДМИН</h3>
                            <div class="price">ДОСТУП <span class="period">/по паролю</span></div>
                        </div>
                        <ul class="features">
                            <li><i class="fas fa-check green"></i> Полный доступ к Grafana Dashboard</li>
                            <li><i class="fas fa-check green"></i> Глубокие метрики Prometheus</li>
                            <li><i class="fas fa-check green"></i> Мониторинг CPU/RAM/Диска</li>
                            <li><i class="fas fa-check green"></i> Логи игрового сервера в реальном времени</li>
                            <li><i class="fas fa-check green"></i> Управление бэкапами мира</li>
                        </ul>
                        <div class="pricing-footer">Требуется авторизация</div>
                    </div>
                    <div class="pricing-card">
                        <div class="pricing-header">
                            <h3>СИСТЕМНЫЙ</h3>
                            <div class="price">СЕРВЕР <span class="period">/root</span></div>
                        </div>
                        <ul class="features">
                            <li><i class="fas fa-check green"></i> Полный root-доступ к Ubuntu Server</li>
                            <li><i class="fas fa-check green"></i> Управление Docker-контейнерами</li>
                            <li><i class="fas fa-check green"></i> Настройка сетевого фаервола</li>
                            <li><i class="fas fa-check green"></i> Просмотр системных логов (journalctl)</li>
                            <li><i class="fas fa-check green"></i> SSH-ключи и безопасность</li>
                        </ul>
                        <div class="pricing-footer">Только для владельца сервера</div>
                    </div>
                </div>
            </section>

            <!-- Новости / Changelog -->
            <section class="section">
                <h2 class="section-title"><i class="fas fa-history"></i> Новости (Changelog)</h2>
                <div class="changelog">
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-date">[16-01-2026]</span>
                            <span class="log-type info">:: INFO</span>
                        </div>
                        <div class="log-title">Запущен новый веб-интерфейс мониторинга</div>
                        <p class="log-desc">Полностью переработан сайт в стиле терминала. Добавлена интеграция со всеми сервисами.</p>
                    </div>
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-date">[15-01-2026]</span>
                            <span class="log-type update">:: UPDATE</span>
                        </div>
                        <div class="log-title">Обновлена панель Grafana</div>
                        <p class="log-desc">Добавлены новые дашборды для мониторинга производительности Minecraft-сервера.</p>
                    </div>
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-date">[14-01-2026]</span>
                            <span class="log-type warn">:: WARN</span>
                        </div>
                        <div class="log-title">Плановые работы по обслуживанию</div>
                        <p class="log-desc">17 января с 03:00 до 04:00 МСК возможны кратковременные перебои в работе.</p>
                    </div>
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-date">[13-01-2026]</span>
                            <span class="log-type feature">:: NEW FEATURE</span>
                        </div>
                        <div class="log-title">Добавлен Node Exporter</div>
                        <p class="log-desc">Теперь доступен мониторинг системных метрик: температура CPU, использование диска, нагрузка сети.</p>
                    </div>
                    <div class="log-entry">
                        <div class="log-header">
                            <span class="log-date">[10-01-2026]</span>
                            <span class="log-type info">:: INFO</span>
                        </div>
                        <div class="log-title">Обновление ядра сервера</div>
                        <p class="log-desc">Установлены последние обновления безопасности для Ubuntu Server 22.04.</p>
                    </div>
                </div>
            </section>
        </main>

        <!-- Футер с ссылками -->
        <footer class="terminal-footer">
            <div class="footer-line"></div>
            <div class="footer-content">
                <div class="footer-section">
                    <h4><i class="fas fa-shield-alt"></i> Партнёры & Технологии</h4>
                    <div class="tech-tags">
                        <span class="tech-tag">Ubuntu Server</span>
                        <span class="tech-tag">Docker</span>
                        <span class="tech-tag">NGINX</span>
                        <span class="tech-tag">Prometheus</span>
                        <span class="tech-tag">Grafana</span>
                        <span class="tech-tag">PaperMC</span>
                    </div>
                </div>
                <div class="footer-section">
                    <h4><i class="fas fa-code"></i> Контакты</h4>
                    <p>По вопросам работы сервера:</p>
                    <p><i class="fab fa-discord"></i> Discord: <code>server#0000</code></p>
                    <p><i class="fas fa-envelope"></i> Telegram: <code>@server_admin</code></p>
                </div>
                <div class="footer-copyright">
                    <p>root@minecraft:~# Сервисы, которые так нужны вам © 2026</p>
                    <p>// Хостинг и инфраструктура: домашний сервер на Ubuntu, 4GB RAM, 2 CPU cores</p>
                </div>
            </div>
        </footer>
    </div>

    <!-- Подключим отдельный JS файл -->
    <script src="/js/app.js"></script>
</body>
</html>
```