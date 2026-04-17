Вот весь текст с изменёнными строками (убраны ссылки, команды приведены к единому формату):


**Топология сети**:

Во второй части лабораторной работы два канала между S1 и S2 не могли работать одновременно. Для полноценного использования пропускной способности обоих каналов между S1 и S2 необходимо настроить агрегирование каналов Ethernet.

![Image title](https://ssisk.tcs.miet.ru/resources/lr3/4.png)

Используемое оборудование

- **Коммутаторы**: S5700

**План работы**:

1. Настройка агрегирования каналов вручную.
2. Настройка агрегирования каналов в режиме LACP.
3. Изменение параметров для определения активных каналов.
4. Изменение режима балансировки нагрузки.

---

### **Шаг 1. Настройте агрегирование каналов вручную**

Создайте Eth-Trunk.

```
[S1]interface Eth-Trunk 1
[S2]interface Eth-Trunk 1
```

Команда **`interface eth-trunk`** позволяет перейти в режим настройки существующего Eth-Trunk или создать Eth-Trunk. Цифра 1, используемая в примере, означает номер интерфейса.

Сконфигурируйте режим агрегирования каналов для Eth-Trunk.

```
[S1-Eth-Trunk1]mode manual load-balance
```

Команда **`mode`** позволяет настроить режим Eth-Trunk. По умолчанию используется режим ручной балансировки нагрузки. Таким образом, предыдущую операцию выполнять не требуется, она приводится только для наглядности.

Добавьте порт в Eth-Trunk.

```
[S1]interface GigabitEthernet 0/0/10
[S1-GigabitEthernet0/0/10]eth-trunk 1
Info: This operation may take a few seconds. Please wait for a moment...done.
[S1-GigabitEthernet0/0/10]quit
[S1]interface GigabitEthernet 0/0/11
[S1-GigabitEthernet0/0/11]eth-trunk 1
Info: This operation may take a few seconds. Please wait for a moment...done.
[S1-GigabitEthernet0/0/11]quit
[S1]interface GigabitEthernet 0/0/12
[S1-GigabitEthernet0/0/12]eth-trunk 1
Info: This operation may take a few seconds. Please wait for a moment...done.
[S1-GigabitEthernet0/0/12]quit
```

Для добавления определенного порта в Eth-Trunk можно перейти в режим настройки его интерфейса и выполнить операцию. Для добавления нескольких портов в Eth-Trunk можно выполнить команду **`trunkport`** в режиме настройки интерфейса Eth-Trunk.

```
[S2]interface Eth-Trunk 1
[S2-Eth-Trunk1]trunkport GigabitEthernet 0/0/10 to 0/0/12
Info: This operation may take a few seconds. Please wait for a moment...done.
```

При добавлении физических портов в Eth-Trunk необходимо учитывать следующие нюансы:

- Агрегированный канал Eth-Trunk может содержать максимум 8 портов-участников.
- Eth-Trunk нельзя добавить к другому Eth-Trunk.
- Порт можно добавить только к одному Eth-Trunk. Чтобы добавить порт к другому Eth-Trunk, сначала необходимо удалить его из исходного.
- Удаленные порты, напрямую подключенные к локальным портам-участникам Eth-Trunk, также должны быть добавлены в Eth-Trunk, в противном случае устройства не смогут взаимодействовать.
- Такие параметры, как количество физических портов, скорость порта и дуплексный режим, должны совпадать на обоих концах канала Eth-Trunk.

Выведите на экран статус Eth-Trunk.

```
[S1]disp eth-trunk 1
Eth-Trunk1's state information is:
WorkingMode: NORMAL         Hash arithmetic: According to SIP-XOR-DIP
Least Active-linknumber: 1  Max Bandwidth-affected-linknumber: 8
Operate status: up          Number Of Up Port In Trunk: 3
--------------------------------------------------------------------------------
PortName                      Status      Weight
GigabitEthernet0/0/10         Up          1
GigabitEthernet0/0/11         Up          1
GigabitEthernet0/0/12         Up          1
```

---

### **Шаг 2. Настройте агрегирование каналов LACP**

Удалите порты-участники из Eth-Trunk.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]undo trunkport GigabitEthernet 0/0/10 to 0/0/12
Info: This operation may take a few seconds. Please wait for a moment...done.
[S2]interface Eth-Trunk 1
[S2-Eth-Trunk1]undo trunkport GigabitEthernet 0/0/10 to 0/0/12
Info: This operation may take a few seconds. Please wait for a moment...done.
```

Измените режим агрегирования. Перед изменением режима работы Eth-Trunk убедитесь, что в Eth-Trunk нет портов-участников.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]mode lacp
```

Команда **`mode lacp`** позволяет установить LACP в качестве рабочего режима Eth-Trunk.

Примечание

В некоторых версиях для этого используется команда **`mode lacp-static`**.

```
[S2]interface Eth-Trunk 1
[S2-Eth-Trunk1]mode lacp
```

Добавьте порт в Eth-Trunk.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]trunkport GigabitEthernet 0/0/10 to 0/0/12
Info: This operation may take a few seconds. Please wait for a moment...done.
[S2]interface Eth-Trunk 1
[S2-Eth-Trunk1]trunkport GigabitEthernet 0/0/10 to 0/0/12
Info: This operation may take a few seconds. Please wait for a moment...done.
```

Выведите на экран статус Eth-Trunk.

```
[S1-Eth-Trunk1]disp eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC
Preempt Delay: Disabled     Hash arithmetic: According to SIP-XOR-DIP
System Priority: 32768      System ID: 4c1f-ccfb-2719
Least Active-linknumber: 1  Max Active-linknumber: 8
Operate status: up          Number Of Up Port In Trunk: 3
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/10  Selected 1GE      32768   11     305     10111100  1
GigabitEthernet0/0/11  Selected 1GE      32768   12     305     10111100  1
GigabitEthernet0/0/12  Selected 1GE      32768   13     305     10111100  1

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/10  32768    4c1f-ccb5-6af4  32768   11     305     10111100
GigabitEthernet0/0/11  32768    4c1f-ccb5-6af4  32768   12     305     10111100
GigabitEthernet0/0/12  32768    4c1f-ccb5-6af4  32768   13     305     10111100
```

---

### **Шаг 3. Измените приоритет LACP на S1**

В обычных условиях в состоянии передачи данных должны находиться только GigabitEthernet0/0/11 и GigabitEthernet0/0/12, а GigabitEthernet0/0/10 должен использоваться в качестве резервного порта. Когда количество активных портов становится меньше 2, Eth-Trunk отключается.

Установите приоритет LACP для S1, чтобы сделать S1 активным устройством.

```
[S1]lacp priority 100
```

Настройте высокий приоритет портам GigabitEthernet0/0/11 и GigabitEthernet0/0/12.

```
[S1]interface GigabitEthernet 0/0/10
[S1-GigabitEthernet0/0/10]lacp priority 40000
```

В режиме LACP пакеты LACPDU (LACP Data Unit) передаются и принимаются обеими сторонами группы агрегирования каналов.

Сначала выбирается активный инициатор:

1. Выполняется сравнение полей приоритета системы. По умолчанию используется значение приоритета 32768. Чем меньше значение, тем выше приоритет. Сторона с более высоким приоритетом выбирается в качестве активного инициатора LACP.
2. При одинаковых приоритетах активным инициатором становится сторона с меньшим MAC-адресом.

После того как активный инициатор выбран, устройства на обеих сторонах выбирают активные порты в соответствии с настройками приоритета порта на активном инициаторе.

Задайте верхний и нижний пороги количества активных портов.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]max active-linknumber 2
[S1-Eth-Trunk1]least active-linknumber 2
```

Пропускная способность и статус Eth-Trunk зависят от количества активных портов. Под пропускной способностью Eth-Trunk подразумевается общая пропускная способность всех портов-участников в состоянии Up. Для того, чтобы стабилизировать статус и пропускную способность Eth-Trunk, а также сократить влияние частых изменений статусов каналов-участников, можно настроить следующие пороговые значения:

- **Нижний порог**: при сокращении количества активных портов ниже этого порога Eth-Trunk отключается. Порог определяет минимальную пропускную способность Eth-Trunk и настраивается с помощью команды **`least active-linknumber`**.
- **Верхний порог**: если количество активных портов достигает этого порогового значения, пропускная способность Eth-Trunk не увеличивается, даже при увеличении числа каналов в состоянии Up. Верхний порог обеспечивает доступность сети и настраивается с помощью команды **`max active-linknumber`**.

Включите preempt-mode.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]lacp preempt enable
```

В режиме LACP при выходе из строя активного канала система выбирает резервный канал с наивысшим приоритетом, чтобы заменить неисправный. Если включена функция внеочередного занятия линии, то после восстановления неисправный канал может снова получить статус активного канала, если он имеет более высокий приоритет, чем резервный канал. Функцию внеочередного занятия линии можно включить с помощью команды lacp preempt enable. По умолчанию эта функция отключена.

Выведите на экран статус текущего Eth-Trunk.

```
[S1]disp eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC
Preempt Delay Time: 30      Hash arithmetic: According to SIP-XOR-DIP
System Priority: 100        System ID: 4c1f-ccfb-2719
Least Active-linknumber: 2  Max Active-linknumber: 2
Operate status: up          Number Of Up Port In Trunk: 2
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/10  Unselect 1GE      40000   11     305     10100000  1
GigabitEthernet0/0/11  Selected 1GE      32768   12     305     10111100  1
GigabitEthernet0/0/12  Selected 1GE      32768   13     305     10111100  1

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/10  32768    4c1f-ccb5-6af4  32768   11     305     10110000
GigabitEthernet0/0/11  32768    4c1f-ccb5-6af4  32768   12     305     10111100
GigabitEthernet0/0/12  32768    4c1f-ccb5-6af4  32768   13     305     10111100
```

Отключите GigabitEthernet0/0/12, чтобы смоделировать неисправность канала. Выведите на экран статус Eth-Trunk.

```
[S1]disp eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC
Preempt Delay Time: 30      Hash arithmetic: According to SIP-XOR-DIP
System Priority: 100        System ID: 4c1f-ccfb-2719
Least Active-linknumber: 2  Max Active-linknumber: 2
Operate status: up          Number Of Up Port In Trunk: 2
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/10  Selected 1GE      40000   11     305     10111100  1
GigabitEthernet0/0/11  Selected 1GE      32768   12     305     10111100  1
GigabitEthernet0/0/12  Unselect 1GE      32768   13     305     10100010  1

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/10  32768    4c1f-ccb5-6af4  32768   11     305     10111100
GigabitEthernet0/0/11  32768    4c1f-ccb5-6af4  32768   12     305     10111100
GigabitEthernet0/0/12  0        0000-0000-0000  0       0      0       10100011
```

Отключите GigabitEthernet 0/0/11, чтобы смоделировать неисправность канала. Выведите на экран статус Eth-Trunk.

```
[S1]disp eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC
Preempt Delay Time: 30      Hash arithmetic: According to SIP-XOR-DIP
System Priority: 100        System ID: 4c1f-ccfb-2719
Least Active-linknumber: 2  Max Active-linknumber: 2
Operate status: down        Number Of Up Port In Trunk: 0
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/10  Unselect 1GE      40000   11     305     10100000  1
GigabitEthernet0/0/11  Unselect 1GE      32768   12     305     10100010  1
GigabitEthernet0/0/12  Unselect 1GE      32768   13     305     10100010  1

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/10  32768    4c1f-ccb5-6af4  32768   11     305     10110000
GigabitEthernet0/0/11  0        0000-0000-0000  0       0      0       10100011
GigabitEthernet0/0/12  0        0000-0000-0000  0       0      0       10100011
```

В качестве нижнего порога количества активных каналов настроено значение 2. Таким образом, Eth-Trunk отключен. Хотя GigabitEthernet0/0/10 стал активным, он все еще имеет статус Unselect.

---

### **Шаг 4. Измените режим балансировки нагрузки**

Включите порты, отключенные на предыдущем шаге. Подождите около 30 секунд и проверьте статус Eth-Trunk 1.

```
[S1]disp eth-trunk 1
Eth-Trunk1's state information is:
Local:
LAG ID: 1                   WorkingMode: STATIC
Preempt Delay Time: 30      Hash arithmetic: According to SIP-XOR-DIP
System Priority: 100        System ID: 4c1f-ccfb-2719
Least Active-linknumber: 2  Max Active-linknumber: 2
Operate status: up          Number Of Up Port In Trunk: 2
--------------------------------------------------------------------------------
ActorPortName          Status   PortType PortPri PortNo PortKey PortState Weight
GigabitEthernet0/0/10  Unselect 1GE      40000   11     305     10100000  1
GigabitEthernet0/0/11  Selected 1GE      32768   12     305     10111100  1
GigabitEthernet0/0/12  Selected 1GE      32768   13     305     10111100  1

Partner:
--------------------------------------------------------------------------------
ActorPortName          SysPri   SystemID        PortPri PortNo PortKey PortState
GigabitEthernet0/0/10  32768    4c1f-ccb5-6af4  32768   11     305     10110000
GigabitEthernet0/0/11  32768    4c1f-ccb5-6af4  32768   12     305     10111100
GigabitEthernet0/0/12  32768    4c1f-ccb5-6af4  32768   13     305     10111100
```

Функция внеочередного занятия линии включена на Eth-Trunk. Таким образом, GigabitEthernet0/0/11 и GigabitEthernet0/0/12 становятся активными, потому что имеют более высокий приоритет, чем GigabitEthernet0/0/10. В результате GigabitEthernet0/0/10 получает статус Unselect. Кроме того, для обеспечения стабильности канала время внеочередного занятия линии по умолчанию составляет 30 секунд. Таким образом, внеочередное занятие линии происходит через 30 секунд после включения портов.

Измените режим балансировки нагрузки Eth-Trunk на балансировку нагрузки на основе IP-адреса назначения.

```
[S1]interface Eth-Trunk 1
[S1-Eth-Trunk1]load-balance dst-ip
```

Чтобы обеспечить правильную балансировку нагрузки между физическими каналами Eth-Trunk и избежать перегрузки каналов, настройте режим балансировки нагрузки Eth-Trunk с помощью команды **`load-balance`**.

Балансировка нагрузки работает только для исходящего трафика. Поэтому режимы балансировки нагрузки для портов на разных сторонах виртуального канала могут отличаться.