# Huawei VLAN

**Топология сети**:
![[Pasted image 20260410111556.png]]
Компании необходимо разделить сеть на несколько VLAN для удовлетворения служебных требований.

Сеть `VLAN 10` должна обеспечивать более высокий уровень безопасности, поэтому в нее можно добавить только специальные ПК.

Используемое оборудование

- **Маршрутизаторы**: AR2220
- **Коммутаторы**: S5700

**План работы**:

1. Создание VLAN.
2. Конфигурирование VLAN на основе портов.
3. Конфигурирование VLAN на основе MAC-адресов.

---

### **Шаг 1. Задайте устройствам имена и настройте IP-адресацию**

Установите для R1 и R3 IP-адреса `10.1.2.1/24` и `10.1.10.1/24` соответственно.
```
[R1]interface GigabitEthernet0/0/1
[R1-GigabitEthernet0/0/1]ip address 10.1.2.1 24
[R3]interface GigabitEthernet0/0/2
[R3-GigabitEthernet0/0/2]ip address 10.1.10.1 24
```

Создайте `VLAN 3` на S3 и S4.

```
[S3]vlan 3
[S4]vlan 3
```

Настройте порты на S3 и S4 в качестве портов доступа и назначьте их в соответствующие VLAN.

```
[S3]interface GigabitEthernet0/0/1
[S3-GigabitEthernet0/0/1]port link-type access
[S3-GigabitEthernet0/0/1]port default vlan 3
[S3-GigabitEthernet0/0/1]quit
[S4]interface GigabitEthernet0/0/2
[S4-GigabitEthernet0/0/2]port link-type access
[S4-GigabitEthernet0/0/2]port default vlan 3
[S4-GigabitEthernet0/0/2]quit
```

Создайте интерфейсы `VLANIF` и настройте IP-адреса.

```
[S3]interface Vlanif 3
[S3-Vlanif3]ip address 10.1.3.1 24
[S4]interface Vlanif 3
[S4-Vlanif3]ip address 10.1.3.2 24
```

С помощью команды **`interface vlanif`** _`vlan-id`_ можно создать интерфейс `VLANIF` и перейти в режим его конфигурирования.

---

### **Шаг 2. Создайте VLAN на S1 и S2**

```
[S1]vlan batch 2 to 3 10
Info: This operation may take a few seconds. Please wait for a moment...done.
VLANs 2, 3, and 10 are created successfully.
```

С помощью команды **`vlan`** _`vlan-id`_ можно создать VLAN и перейти в режим его конфигурирования.

Команда **`vlan batch`** позволяет создавать сразу несколько VLAN.

```
[S2]vlan batch 2 to 3 10
```

---

### **Шаг 3. Настройте VLAN на портах**

Настройте пользовательские порты на S1 и S2 в качестве портов доступа и назначьте их в соответствующие VLAN.

```
[S1]interface GigabitEthernet0/0/1
[S1-GigabitEthernet0/0/1]port link-type access
[S1-GigabitEthernet0/0/1]port default vlan 2
[S1-GigabitEthernet0/0/1]quit
[S1]interface GigabitEthernet0/0/13
[S1-GigabitEthernet0/0/13]port link-type access
[S1-GigabitEthernet0/0/13]port default vlan 3
[S1-GigabitEthernet0/0/13]quit
[S2]interface GigabitEthernet0/0/14
[S2-GigabitEthernet0/0/14]port link-type access
[S2-GigabitEthernet0/0/14]port default vlan 3
[S2-GigabitEthernet0/0/14]quit
```

С помощью команды **`port link-type`** _`{access | hybrid | trunk}`_ можно задать тип интерфейса, который может быть Access, Trunk или Hybrid.

Команда **`port default vlan`** _`vlan-id`_ позволяет настроить VLAN по умолчанию для интерфейса и назначить интерфейс в эту сеть VLAN.

Настройте порты, соединяющие S1 и S2, в качестве магистральных портов и разрешите прохождение только пакетов из `VLAN 2` и `VLAN 3`.

```
[S1]interface GigabitEthernet0/0/10
[S1-GigabitEthernet0/0/10]port link-type trunk
[S1-GigabitEthernet0/0/10]port trunk allow-pass vlan 2 3
[S1-GigabitEthernet0/0/10]undo port trunk allow-pass vlan 1
[S2]interface GigabitEthernet0/0/10
[S2-GigabitEthernet0/0/10]port link-type trunk
[S2-GigabitEthernet0/0/10]port trunk allow-pass vlan 2 3
[S2-GigabitEthernet0/0/10]undo port trunk allow-pass vlan 1
```

Команда **`port trunk allow-pass`** _`vlan`_ позволяет назначить магистральный порт в определенные сети VLAN.

Команда **`undo port trunk allow-pass`** _`vlan`_ позволяет удалить магистральный порт из определенных сетей VLAN. По умолчанию VLAN 1 находится в списке разрешенных сетей. Если VLAN 1 не используется, то эту сеть необходимо удалить в целях безопасности.

---

### **Шаг 4. Настройте привязку VLAN к MAC-адресам**

Как показано на рисунке с топологией, R3 имитирует специальный служебный ПК. **Допустим**, что данный ПК имеет MAC-адрес `a008-6fe1-0c46`.

Предполагается, что ПК будет подключаться к сети через любой из портов GigabitEthernet0/0/1, GigabitEthernet0/0/2 и GigabitEthernet0/0/3 на S2 и передавать данные через `VLAN 10`.

Настройте на S2 привязку MAC-адреса R3 к `VLAN 10`.

```
[S2]vlan 10
[S2-vlan10]mac-vlan mac-address a008-6fe1-0c46
```

Команда **`mac-vlan`** _`mac-address`_ позволяет установить привязку MAC-адреса к VLAN. Принадлежность к VLAN зависит от MAC-адресов источника пакетов. Этот метод назначения VLAN не зависит от местоположения пользователя, обеспечивая более высокий уровень безопасности и гибкости.

На портах доступа и магистральных портах назначение VLAN на основе MAC-адресов можно использовать только в том случае, если VLAN соответствует PVID. Поэтому рекомендуется использовать назначение VLAN на основе MAC-адресов на гибридных портах для получения нетегированных пакетов нескольких VLAN.

Настройте GigabitEthernet0/0/1, GigabitEthernet0/0/2 и GigabitEthernet0/0/3 на S2 в качестве гибридных портов и разрешите прохождение пакетов из `VLAN 10`.

```
[S2]interface GigabitEthernet0/0/1
[S2-GigabitEthernet0/0/1]port link-type hybrid
[S2-GigabitEthernet0/0/1]port hybrid untagged vlan 10
[S2-GigabitEthernet0/0/1]quit
[S2]interface GigabitEthernet0/0/2
[S2-GigabitEthernet0/0/2]port link-type hybrid
[S2-GigabitEthernet0/0/2]port hybrid untagged vlan 10
[S2-GigabitEthernet0/0/2]quit
[S2]interface GigabitEthernet0/0/3
[S2-GigabitEthernet0/0/3]port link-type hybrid
[S2-GigabitEthernet0/0/3]port hybrid untagged vlan 10
[S2-GigabitEthernet0/0/3]quit
```

Команда **`port hybrid untagged`** _`vlan`_ позволяет назначить гибридный порт в определенные сети VLAN, чтобы передавать нетегированные кадры.

Настройте на портах, соединяющих S1 и S2, разрешение на прохождение пакетов из `VLAN 10`. Порты должны разрешать прохождение тегированных кадров из нескольких VLAN. Следовательно, порты нужно настроить в качестве магистральных портов.

```
[S1]interface GigabitEthernet0/0/10
[S1-GigabitEthernet0/0/10]port trunk allow-pass vlan 10
[S1-GigabitEthernet0/0/10]quit
[S2]interface GigabitEthernet0/0/10
[S2-GigabitEthernet0/0/10]port trunk allow-pass vlan 10
[S2-GigabitEthernet0/0/10]quit
```

Настройте S2 и включите назначение VLAN на основе MAC-адресов на GE0/0/1, GE0/0/2 и GE0/0/3. Чтобы включить на порте передачу пакетов на основе привязки между MAC-адресом и VLAN, необходимо выполнить команду `mac-vlan enable`.

```
[S2]interface GigabitEthernet0/0/1
[S2-GigabitEthernet0/0/1]mac-vlan enable
[S2-GigabitEthernet0/0/1]quit
[S2]interface GigabitEthernet0/0/2
[S2-GigabitEthernet0/0/2]mac-vlan enable
[S2-GigabitEthernet0/0/2]quit
[S2]interface GigabitEthernet0/0/3
[S2-GigabitEthernet0/0/3]mac-vlan enable
[S2-GigabitEthernet0/0/3]quit
```

---

### **Шаг 5. Выведите на экран информацию о конфигурации**

Выведите на экран информацию о VLAN на коммутаторе.

```
[S1]display vlan
```

Команда **`display vlan`** позволяет вывести на экран информацию о сетях VLAN.

С помощью команды **`display vlan`** _`vlan-id`_ **`verbose`** можно вывести на экран подробную информацию определенной VLAN, включая идентификатор, тип, описание и состояние VLAN, порты VLAN и режим, в котором осуществляется назначение портов в VLAN.

Выведите на экран конфигурацию назначения VLAN на основе MAC-адресов, имеющуюся на коммутаторе.

```
[S2]display mac-vlan vlan 10
```

Команда **`display mac-vlan`** позволяет вывести на экран конфигурацию назначения VLAN на основе MAC-адресов.