Вот весь текст с изменёнными строками (убраны ссылки, команды приведены к единому формату):

**Топология сети**:
![[Pasted image 20260410112038.png]]
В целях повышения сетевой доступности, компании необходимо создать резервные каналы в своей коммутируемой сети. Кроме того, необходимо использовать протокол STP, чтобы предотвратить образование петель на резервных каналах, вызывающих широковещательный шторм и нестабильность таблиц MAC-адресов.

![Image title](https://ssisk.tcs.miet.ru/resources/lr3/2.png)

Используемое оборудование

- **Коммутаторы**: S5700

**План работы**:

1. Включение STP.
2. Изменение приоритетов, чтобы контролировать выбор корневого моста.
3. Изменение параметров порта, чтобы определить роль порта.
4. Изменение версии протокола на RSTP.
5. Настройка граничных портов.

---

### **Шаг 1. Включите STP**

```
<S1>system-view
Enter system view, return user view with Ctrl+Z.
[S1]stp enable
```

Измените режим на STP.

```
[S1]stp mode stp
Info: This operation may take a few seconds. Please wait for a moment...done.
```

С помощью команды **`stp mode`** _`{mstp | rstp | stp}`_ можно установить режим работы протокола связующего дерева на коммутаторе. По умолчанию коммутатор работает в режиме MSTP.

```
[S2]stp mode stp
Info: This operation may take a few seconds. Please wait for a moment...done.
[S3]stp mode stp
Info: This operation may take a few seconds. Please wait for a moment...done.
[S4]stp mode stp
Info: This operation may take a few seconds. Please wait for a moment...done.
```

Выведите на экран статус связующего дерева. В данном случае для примера используется S1.

```
[S1]disp stp
-------[CIST Global Info][Mode STP]-------
CIST Bridge         :32768.4c1f-cc44-4163
Config Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
Active Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
CIST Root/ERPC      :32768.4c1f-cc44-4163 / 0
CIST RegRoot/IRPC   :32768.4c1f-cc44-4163 / 0
CIST RootPortId     :0.0
BPDU-Protection     :Disabled
TC or TCN received  :15
TC count per hello  :0
STP Converge Mode   :Normal
Time since last TC  :0 days 0h:0m:23s
Number of TC        :19
Last TC occurred    :GigabitEthernet0/0/14
```

Выведенная информация также включает данные состояния порта, которые не были включены в командный вывод.

Выведите на экран краткую информацию о связующем дереве на каждом коммутаторе.

```
[S1]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/10       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/11       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/13       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/14       DESI  FORWARDING      NONE
[S2]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/10       ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/11       ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/13       ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/14       DESI  FORWARDING      NONE
[S3]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/2        DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/3        DESI  FORWARDING      NONE
[S4]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/2        ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/3        ALTE  DISCARDING      NONE
```

На основе идентификатора корневого моста и информации о порте каждого коммутатора текущая топология выглядит следующим образом:

![Image title](https://ssisk.tcs.miet.ru/resources/lr3/3.png)

Примечание

Данная топология приводится исключительно в справочных целях, поэтому может не совпадать с фактической топологией связующего дерева в лабораторной среде.

---

### **Шаг 2. Измените приоритеты мостов**

Измените параметры устройств, чтобы сделать S2 корневым мостом, а S1 – резервным корневым мостом.

Измените приоритеты мостов S1 и S2.

```
[S2]stp root primary
[S1]stp root secondary
```

Так как корневой мост играет очень важную роль в сети, то в качестве него обычно выбирается коммутатор с высокой производительностью и высоким уровнем сетевой иерархии. Однако такое устройство может иметь невысокий приоритет. Поэтому необходимо установить коммутатору высокий приоритет, чтобы он мог быть выбран в качестве корневого моста.

Команда **`stp root primary`** позволяет задать коммутатор в качестве корневого. В этом случае коммутатор получит приоритет в связующем дереве, равный 0, и его нельзя будет изменить.

Команда **`stp root secondary`** позволяет задать коммутатор в качестве резервного корневого моста. В этом случае коммутатор получит приоритет, равный 4096, и его нельзя будет изменить.

Выведите на экран статус STP на S2.

```
[S2]disp stp
-------[CIST Global Info][Mode STP]-------
CIST Bridge         :0    .4c1f-ccbc-796a
Config Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
Active Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
CIST Root/ERPC      :0    .4c1f-ccbc-796a / 0
CIST RegRoot/IRPC   :0    .4c1f-ccbc-796a / 0
CIST RootPortId     :0.0
BPDU-Protection     :Disabled
CIST Root Type      :Primary root
TC or TCN received  :917
TC count per hello  :0
STP Converge Mode   :Normal
Time since last TC  :0 days 0h:2m:21s
Number of TC        :38
Last TC occurred    :GigabitEthernet0/0/14
```

В этом случае идентификатор моста S2 совпадает с идентификатором корневого моста, а стоимость корневого маршрута равна 0, что указывает на то, что S2 является корневым мостом текущей сети.

Выведите на экран краткую информацию о статусе STP на всех устройствах и посмотрите как изменилась топология связующего дерева.

```
[S1]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/10       ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/11       ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/13       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/14       DESI  FORWARDING      NONE
[S2]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/10       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/11       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/13       DESI  FORWARDING      NONE
  0    GigabitEthernet0/0/14       DESI  FORWARDING      NONE
[S3]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/3        DESI  FORWARDING      NONE
[S4]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/3        ALTE  DISCARDING      NONE
```

---

### **Шаг 3. Измените параметры S4**

Измените параметры коммутатора S4, чтобы назначить порт GigabitEthernet0/0/1 корневым портом.

Выведите на экран информацию STP на S4.

```
[S4]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/3        ALTE  DISCARDING      NONE
[S4]disp stp
-------[CIST Global Info][Mode STP]-------
CIST Bridge         :32768.4c1f-cccf-4ed7
Config Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
Active Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
CIST Root/ERPC      :0    .4c1f-ccbc-796a / 20000
CIST RegRoot/IRPC   :32768.4c1f-cccf-4ed7 / 0
CIST RootPortId     :128.2
BPDU-Protection     :Disabled
TC or TCN received  :1269
TC count per hello  :0
STP Converge Mode   :Normal
Time since last TC  :0 days 0h:4m:53s
Number of TC        :41
Last TC occurred    :GigabitEthernet0/0/2
```

Стоимость маршрута от S4 до S2 имеет значение 20000.

Измените стоимость STP порта GigabitEthernet 0/0/2 коммутатора S4 на 50000.

```
[S4]interface GigabitEthernet 0/0/2
[S4-GigabitEthernet0/0/2]stp cost 50000
```

Выведите на экран краткую информацию о статусе STP.

```
[S4]disp stp br
MSTID  Port                        Role  STP State     Protection
  0    GigabitEthernet0/0/1        ROOT  FORWARDING      NONE
  0    GigabitEthernet0/0/2        ALTE  DISCARDING      NONE
  0    GigabitEthernet0/0/3        ALTE  DISCARDING      NONE
```

Порт GigabitEthernet0/0/1 на S4 стал корневым портом.

---

### **Шаг 4. Измените режим связующего дерева на RSTP**

```
[S1]stp mode rstp
Info: This operation may take a few seconds. Please wait for a moment...done
```

Измените режим связующего дерева на всех устройствах.

Выведите на экран статус связующего дерева. В данном случае для примера используется S1.

```
[S1]disp stp
-------[CIST Global Info][Mode RSTP]-------
CIST Bridge         :4096 .4c1f-cc44-4163
Config Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
Active Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
CIST Root/ERPC      :0    .4c1f-ccbc-796a / 20000
CIST RegRoot/IRPC   :4096 .4c1f-cc44-4163 / 0
CIST RootPortId     :128.10
BPDU-Protection     :Disabled
CIST Root Type      :Secondary root
TC or TCN received  :265
TC count per hello  :0
STP Converge Mode   :Normal
Time since last TC  :0 days 0h:0m:6s
Number of TC        :49
Last TC occurred    :GigabitEthernet0/0/10
```

После изменения режима топология связующего дерева не изменилась.

---

### **Шаг 5. Настройте граничные порты**

Порты GigabitEthernet 0/0/10-0/0/24 коммутатора S3 подключены только к терминалам, поэтому их необходимо настроить в качестве граничных портов.

```
[S3]port-group group-member GigabitEthernet 0/0/10 to GigabitEthernet 0/0/24
[S3-port-group]stp edged-port enable
```

Команда **`stp edged-port enable`** позволяет задать текущий порт в качестве граничного порта. Если после граничный порт получает BPDU, то он перестает быть граничным и STP пересчитывает топологию.
