**Протокол OSPF** был разработан организацией IETF, является **Link-State** протоколом динамической маршрутизации, в основе его работы лежит алгоритм **Дейкстры**.

В настоящее время в сетях IPv4 используется OSPF версии 2 (**RFC2328**). Протокол OSPF имеет следующие преимущества:

1. Многоадресная передача пакетов для снижения нагрузки на коммутаторы, на которых не работает OSPF.
2. Бесклассовая междоменная маршрутизация (Classless Inter-Domain Routing, **CIDR**).
3. Балансировка нагрузки между равноценными маршрутами.
4. Пакетная аутентификация.

В качестве **отчета** по данной части предоставьте скрины её пошагового выполнения.

**Топология сети:**
![[Pasted image 20260410112216.png]]
![Image title](https://ssisk.tcs.miet.ru/resources/lr2/1.png)

Используемое оборудование
- **Маршрутизаторы**: AR2220

**План работы:**

1. Создание процессов OSPF на устройствах и включение OSPF на интерфейсах.
2. Настройка аутентификации OSPF.
3. Настройка анонсирования маршрутов по умолчанию.
4. Управление выбором маршрутов OSPF на основании их стоимости.

---

### **Шаг 1. Настройте основные параметры устройств**

1. Задайте имена устройствам.
2. Настройте IP-адреса для физических интерфейсов согласно таблице 1.1.
3. Настройте IP-адреса для виртуальных интерфейсов согласно таблице 1.2.

Таблица 1.1 – IP-адреса для физических интерфейсов

|Маршрутизатор|Интерфейс|IP-адрес/маска|
|---|---|---|
|R1|GigabitEthernet0/0/0|10.0.12.1/24|
||GigabitEthernet0/0/2|10.0.13.1/24|
|R2|GigabitEthernet0/0/0|10.0.12.2/24|
||GigabitEthernet0/0/1|10.0.23.2/24|
|R3|GigabitEthernet0/0/1|10.0.23.3/24|
||GigabitEthernet0/0/2|10.0.13.3/24|

Таблицы 1.2 – IP-адреса для виртуальных интерфейсов

|Маршрутизатор|Интерфейс|IP-адрес/маска|
|---|---|---|
|R1|LoopBack0|10.0.1.1/32|
|R2|LoopBack0|10.0.1.2/32|
|R3|LoopBack0|10.0.1.3/32|

---

### **Шаг 2. Настройте основные параметры OSPF**

Создайте процесс OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-0-1)[R1]ospf 1`

Настройка параметров OSPF станет возможной только после создания процесса OSPF. OSPF поддерживает несколько независимых процессов на одном устройстве. Обмен маршрутами между различными процессами OSPF осуществляется аналогично обмену маршрутами между разными протоколами маршрутизации. При создании процесса OSPF можно указать идентификатор процесса. Если идентификатор процесса не указан, то по умолчанию используется идентификатор процесса 1.

Создайте область OSPF и укажите интерфейсы, на которых необходимо включить OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-1-1)[R1-ospf-1]area 0`

С помощью команды `area` можно создать область OSPF и перейти в режим конфигурирования области OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-2-1)[R1-ospf-1-area-0.0.0.0]network 10.0.12.1 0.0.0.255 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-2-2)[R1-ospf-1-area-0.0.0.0]network 10.0.13.1 0.0.0.255 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-2-3)[R1-ospf-1-area-0.0.0.0]network 10.0.1.1 0.0.0.0` 

С помощью команды **`network`** _`<network-address>`_ _`<wildcard-mask>`_ можно указать интерфейсы, на которых необходимо включить OSPF. Он будет работать на интерфейсе только при соблюдении следующих двух условий:

1. Длина маски IP-адреса интерфейса должна быть не короче, чем длина маски, указанная в команде `network`. Для OSPF должна использоваться обратная маска. Например, 0.0.0.255 указывает, что длина маски составляет 24 бита.
2. Адрес интерфейса должен находиться в пределах сетевого диапазона, указанного в команде `network`.

В данном примере OSPF можно включить на трех интерфейсах, и все они добавлены в область 0.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-3-1)[R2]ospf   [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-3-2)[R2-ospf-1]area 0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-3-3)[R2-ospf-1-area-0.0.0.0]network 10.0.12.2 0.0.0.0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-3-4)[R2-ospf-1-area-0.0.0.0]network 10.0.23.2 0.0.0.0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-3-5)[R2-ospf-1-area-0.0.0.0]network 10.0.1.2 0.0.0.0`

Если обратная маска в команде `network` включает только нули (0.0.0.0) и IP-адрес интерфейса совпадает с IP-адресом, указанным в команде `network`, то на интерфейсе также будет работать OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-4-1)[R3]ospf   [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-4-2)[R3-ospf-1]area 0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-4-3)[R3-ospf-1-area-0.0.0.0]network 10.0.13.3 0.0.0.0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-4-4)[R3-ospf-1-area-0.0.0.0]network 10.0.23.3 0.0.0.0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-4-5)[R3-ospf-1-area-0.0.0.0]network 10.0.1.3 0.0.0.0`

### **Шаг 3. Выведите на экран статус OSPF**

Выведите на экран информацию о соседях OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-1)[R1]disp ospf peer [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-2) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-3)     OSPF Process 1 with Router ID 10.0.12.1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-4)         Neighbors  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-5) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-6) Area 0.0.0.0 interface 10.0.12.1(GigabitEthernet0/0/0)'s neighbors [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-7) Router ID: 10.0.12.2        Address: 10.0.12.2        [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-8)   State: Full  Mode:Nbr is  Master  Priority: 1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-9)   DR: 10.0.12.2  BDR: 10.0.12.1  MTU: 0     [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-10)   Dead timer due in 32  sec  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-11)   Retrans timer interval: 0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-12)   Neighbor is up for 00:00:16      [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-13)   Authentication Sequence: [ 0 ]  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-14) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-15)         Neighbors  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-16) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-17) Area 0.0.0.0 interface 10.0.13.1(GigabitEthernet0/0/2)'s neighbors [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-18) Router ID: 10.0.23.3        Address: 10.0.13.3        [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-19)   State: Full  Mode:Nbr is  Master  Priority: 1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-20)   DR: 10.0.13.3  BDR: 10.0.13.1  MTU: 0     [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-21)   Dead timer due in 36  sec  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-22)   Retrans timer interval: 5  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-23)   Neighbor is up for 00:00:06      [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-5-24)   Authentication Sequence: [ 0 ]`

Команда `display ospf peer` позволяет вывести на экран информацию о соседях в каждой области OSPF. Информация включает в себя область, к которой принадлежит сосед, идентификатор маршрутизатора соседа, статус соседа, DR и BDR.

Выведите на экран маршруты, полученные от OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-1)[R1]disp ip routing-table protocol ospf [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-2)Route Flags: R - relay, D - download to fib [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-3)------------------------------------------------------------------------------ [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-4)Public routing table : OSPF [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-5)         Destinations : 3        Routes : 4         [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-6) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-7)OSPF routing table status : <Active> [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-8)         Destinations : 3        Routes : 4 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-9) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-10)Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-11) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-12)       10.0.1.2/32  OSPF    10   1           D   10.0.12.2       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-13)       10.0.1.3/32  OSPF    10   1           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-14)      10.0.23.0/24  OSPF    10   2           D   10.0.12.2       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-15)                    OSPF    10   2           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-16) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-17)OSPF routing table status : <Inactive> [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-6-18)         Destinations : 0        Routes : 0`

### **Шаг 4. Настройте аутентификацию OSPF**

Настройте аутентификацию на интерфейсах R1.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-1)[R1]int g 0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-2)[R1-GigabitEthernet0/0/0]ospf authentication-mode md5 1 cipher HCIA-Datacom [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-3)[R1-GigabitEthernet0/0/0]int g 0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-4)[R1-GigabitEthernet0/0/2]ospf authentication-mode md5 1 cipher HCIA-Datacom [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-5)[R1-GigabitEthernet0/0/2]disp this [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-6)[V200R003C00] [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-7)# [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-8)interface GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-9) ip address 10.0.13.1 255.255.255.0  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-10) ospf authentication-mode md5 1 cipher %$%$}l|ZSb4RnL*h(YG^/m6'p-2I%$%$ [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-7-11)#`

При просмотре конфигурации пароль отображается в зашифрованном виде, поскольку в команде указано ключевое слово `cipher`, обеспечивающее шифрование текста.

Выведите на экран соседей OSPF.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-1)[R1]disp ospf peer brief  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-2) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-3)     OSPF Process 1 with Router ID 10.0.12.1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-4)          Peer Statistic Information [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-5) ---------------------------------------------------------------------------- [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-6) Area Id          Interface                        Neighbor id      State     [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-8-7) ----------------------------------------------------------------------------`

На других маршрутизаторах аутентификация не настроена. Следовательно, аутентификация не выполняется, и данные о соседях недоступны.

Настройте аутентификацию на интерфейсах R2. Выведите на экран соседей OSPF на R2.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-1)[R2]display ospf peer brief [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-2) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-3)     OSPF Process 1 with Router ID 10.0.12.2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-4)          Peer Statistic Information [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-5) ---------------------------------------------------------------------------- [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-6) Area Id          Interface                        Neighbor id      State     [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-7) 0.0.0.0          GigabitEthernet0/0/0             10.0.12.1        Full      [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-9-8) ----------------------------------------------------------------------------`

Маршрутизатор R2 установил отношения соседства с маршрутизатором R1.

Настройте аутентификацию области на R3.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-10-1)[R3]ospf  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-10-2)[R3-ospf-1]area 0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-10-3)[R3-ospf-1-area-0.0.0.0]authentication-mode md5 1 cipher HCIA-Datacom`

Выведите на экран соседей OSPF на R3.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-1)[R3-ospf-1]disp ospf peer br [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-2) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-3)     OSPF Process 1 with Router ID 10.0.23.3 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-4)          Peer Statistic Information [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-5) ---------------------------------------------------------------------------- [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-6) Area Id          Interface                        Neighbor id      State     [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-7) 0.0.0.0          GigabitEthernet0/0/1             10.0.12.2        Full         [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-8) 0.0.0.0          GigabitEthernet0/0/2             10.0.12.1        Full         [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-11-9) ----------------------------------------------------------------------------`

Маршрутизатор R3 установил отношения соседства с маршрутизаторами R1 и R2.

### ** Шаг 5. Анонсируйте маршрут по умолчанию**

Предположим, что R1 является пограничным маршрутизатором. Таким образом, маршрутизатор R1 должен анонсировать маршрут по умолчанию.

Анонсируйте маршрут по умолчанию на R1.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-12-1)[R1]ospf   [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-12-2)[R1-ospf-1]default-route-advertise always`

Команда `default-route-advertise` позволяет анонсировать маршрут по умолчанию в OSPF. Если аргумент `always` не указан, маршрут по умолчанию анонсируется другим маршрутизаторам только тогда, когда в таблице маршрутизации локального маршрутизатора есть активные маршруты по умолчанию других протоколов, не OSPF. В данном случае в локальной таблице маршрутизации маршрут по умолчанию отсутствует. Таким образом, необходимо использовать аргумент `always`.

Выведите на экран таблицы IP-маршрутизации R2 и R3.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-1)[R2]disp ip routing-table  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-2)Route Flags: R - relay, D - download to fib [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-3)------------------------------------------------------------------------------ [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-4)Routing Tables: Public [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-5)         Destinations : 15       Routes : 16        [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-6) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-7) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-8) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-9)Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-10) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-11)        0.0.0.0/0   O_ASE   150  1           D   10.0.12.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-12)       10.0.1.1/32  OSPF    10   1           D   10.0.12.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-13)       10.0.1.2/32  Direct  0    0           D   127.0.0.1       LoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-14)       10.0.1.3/32  OSPF    10   1           D   10.0.23.3       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-15)      10.0.12.0/24  Direct  0    0           D   10.0.12.2       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-16)      10.0.12.2/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-17)    10.0.12.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-18)      10.0.13.0/24  OSPF    10   2           D   10.0.12.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-19)                    OSPF    10   2           D   10.0.23.3       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-20)      10.0.23.0/24  Direct  0    0           D   10.0.23.2       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-21)      10.0.23.2/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-22)    10.0.23.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-23)      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-24)      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-25)127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-13-26)255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0`

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-1)[R3]disp ip routing-table  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-2)Route Flags: R - relay, D - download to fib [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-3)------------------------------------------------------------------------------ [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-4)Routing Tables: Public [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-5)         Destinations : 15       Routes : 16        [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-6) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-7)Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-8) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-9)        0.0.0.0/0   O_ASE   150  1           D   10.0.13.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-10)       10.0.1.1/32  OSPF    10   1           D   10.0.13.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-11)       10.0.1.2/32  OSPF    10   1           D   10.0.23.2       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-12)       10.0.1.3/32  Direct  0    0           D   127.0.0.1       LoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-13)      10.0.12.0/24  OSPF    10   2           D   10.0.13.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-14)                    OSPF    10   2           D   10.0.23.2       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-15)      10.0.13.0/24  Direct  0    0           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-16)      10.0.13.3/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-17)    10.0.13.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-18)      10.0.23.0/24  Direct  0    0           D   10.0.23.3       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-19)      10.0.23.3/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-20)    10.0.23.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/1 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-21)      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-22)      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-23)127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-14-24)255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0`

R2 и R3 получили маршрут по умолчанию.

### **Шаг 6. Измените значения стоимости интерфейсов на R1**

Измените значения стоимости интерфейсов на R1 так, чтобы связь между интерфейсами LoopBack0 на R1 и R2 осуществлялась через маршрутизатор R3.

Согласно таблице маршрутизации R1 стоимость маршрута от маршрутизатора R1 до LoopBack0 маршрутизатора R2 равна 1, а стоимость маршрута от R1 к R2 через R3 равна 2. Следовательно, необходимо только установить для стоимости маршрута от маршрутизатора R1 до LoopBack0 маршрутизатора R2 значение больше 2.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-15-1)[R1]interface GigabitEthernet0/0/0   [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-15-2)[R1- GigabitEthernet0/0/0]ospf cost 10`

Выведите на экран таблицу маршрутизации R1.

`[](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-1)[R1]disp ip routing-table  [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-2)Route Flags: R - relay, D - download to fib [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-3)------------------------------------------------------------------------------ [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-4)Routing Tables: Public [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-5)         Destinations : 14       Routes : 14        [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-6) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-7)Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-8) [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-9)       10.0.1.1/32  Direct  0    0           D   127.0.0.1       LoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-10)       10.0.1.2/32  OSPF    10   2           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-11)       10.0.1.3/32  OSPF    10   1           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-12)      10.0.12.0/24  Direct  0    0           D   10.0.12.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-13)      10.0.12.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-14)    10.0.12.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-15)      10.0.13.0/24  Direct  0    0           D   10.0.13.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-16)      10.0.13.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-17)    10.0.13.255/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-18)      10.0.23.0/24  OSPF    10   2           D   10.0.13.3       GigabitEthernet0/0/2 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-19)      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-20)      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-21)127.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0 [](https://ssisk.tcs.miet.ru/labs/lab2/lr/LR2-1/#__codelineno-16-22)255.255.255.255/32  Direct  0    0           D   127.0.0.1       InLoopBack0`