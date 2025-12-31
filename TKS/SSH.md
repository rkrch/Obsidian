port 22/tcp
Secure Shell
## Ключи
```
ssh-keygen -t ed25519
```
pub и privatный
```bash
mkdir .ssh
nano .ssh/authorised_keys
#paste key
chown -R user:group /home/user/.ssh
chmod 700 ~/.ssh
chmod 644 ~./ssh/* 
```

```bash
ssh -i /ключ user@addr
```

Конфиг ssh
```bash
cd /etc/ssh/
nano sshd_config
#например перенести порт, выключить аунт по паролю*
systemctl restart sshd
systemctl status sshd
```

Можно пробросить локалхост 
```bash
ssh -L 9000:localhost:8000 servername
#(если без конфига на подключение без юзернейма то по старинке)
```
## Команды (перенести в линуксоид)
```bash
netstat -tulpn
```

