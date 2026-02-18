[[Маршрут по умолчанию]]
# Команды CISCO Packet Tracer

```
команду написать -> '?' = справка че дальше можно писать (если говно ввел - скажет)

enable - Привилегированный режим
conf t - Режим глобальной конфигурации (configure terminal)
interface FastEthernet 0/0 - Режим конфигурации выбранного интерфейса
```

[[Маршрутизатор (Роутер)]] и [[Коммутатор (Switch)]]:
## Привилегированный режим `enable`

```
show  runnung-config - текущая конфигурация (sh run)
write - сохранение настроек 
show vlan - просмотр состояния VLAN
show ip route - таблица маршрутизации (список сетей, к которым подключен маршрутизатор)
show ip interface brief - вывести список интерфейсов и их состояний
```
## Режим глобальной конфигурации `config`

```
do - запускает команду привилегированного режима (таб не работает)

vlan [N] - создаст vlan
hostname [name] - в cli меняет имя 

username [name] password [pass] - устанавливает логин-пароль, регистр влияет
service password-encryption

line console 0 - переводит в режим конфигурации консольного порта типа
config-line: login local - говорит где брать инфу о логинах-паролях
config-line: exec-timeout 5 - интервал бездействия в минутах до логаута
config-line: logging synchronus - блокает вывод пока пишем команду
config-line: transport input ? - протокол по которому порт открыт
config-line: exit

enable secret [pass] - пароль к привилегированному режиму

no ip domain-lookup - мастхэв для того чтобы не искал неверную команду

#настройка ssh     ДЛЯ РОУТЕРА
ip ssh version [N]
crypto key generate rsa 
aaa new-model - Aунтефикейшн Ауторизейшн энд Аккаунтинг
do wr - вообще почаще юзать сохр-е
line vty 0 15
config-line: transport input ssh
aaa authentication login [name] local
line console 0
config-line: login authentication [same name?]
do wr
 
 
 
interface range fastEthernet 0/1 - 24 - с первого по 24 выбираются
config-if-range: switchport mode access vlan [N]
interface Vlan1
ip address [ip] [маска] 
и с пк будет пинговаться коммутатор
ip domain name [name] 
ip ssh version [n]
crypto key generate rsa 
do wr
line vty 0 15
config-line: transport input ssh
config-line: login local 
//aaa у коммутатора нет


//на роутере стат. марш-я.
R1: ip route [ip_2пк] [маска] [ip*]  - 1пк -> 1 роутер ->* 2 роутер -> 2пк
R0: ip route [сеть] [маска] [ip*]  - 1пк -> 1 роутер *-> 2 роутер -> 2пк
```

## Режим конфигурации интерфейса `interface ____`
```
ip address [ip маршрутизатора] [маска] 
no shutdown - ВАЖНО
do wr

 
```

## PC в CISCO

```
ping [ip]
telnet [ip]
ssh -l [login] [ip]
ipconfig /all
ipv6config /all
tracert 

```