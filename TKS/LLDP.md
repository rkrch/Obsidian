Протокол обнаружения канального уровня - **Link Layer Discovery Protocol**

Протокол, используемый сетевыми устройствами для объявления своей идентичности
Протокол обнаружения канального уровня (LLDP) - это не зависящий от производителя [протокол канального уровня](https://ru.wikibrief.org/wiki/Link_layer), используемый [сетевыми устройствами](https://ru.wikibrief.org/wiki/Network_device) для объявления своей идентичности, возможностей и соседей в [локальной области. сеть](https://ru.wikibrief.org/wiki/Local_area_network) на основе технологии [IEEE 802](https://ru.wikibrief.org/wiki/IEEE_802), в основном [проводной Ethernet](https://ru.wikibrief.org/wiki/Wired_Ethernet).

## Собранная информация

Информация, собранная с помощью LLDP, может храниться в базе данных управления устройством (MIB) и запрашиваться с помощью [Simple Network Management Протокол](https://ru.wikibrief.org/wiki/Simple_Network_Management_Protocol) (SNMP), как указано в [RFC](https://ru.wikibrief.org/wiki/Request_for_Comments) 2922. Топологию сети с поддержкой LLDP можно обнаружить путем сканирования хостов и запросов к этой базе данных. 

**Информация, которую можно получить, включает:**
- Имя и описание системы    
- [Имя и описание порта](https://ru.wikibrief.org/wiki/Port_\(computer_networking\))    
- [Имя VLAN](https://ru.wikibrief.org/wiki/VLAN)    
- IP-адрес управления    
- Возможности системы ( [коммутация](https://ru.wikibrief.org/wiki/Network_switch), [маршрутизация](https://ru.wikibrief.org/wiki/Routing) и т. Д.)    
- [MAC](https://ru.wikibrief.org/wiki/Medium_access_control) / [PHY](https://ru.wikibrief.org/wiki/PHY) информация    
- [мощность MDI](https://ru.wikibrief.org/wiki/Power_over_Ethernet)    
- [агрегация каналов](https://ru.wikibrief.org/wiki/Link_aggregation) 

## Приложения

Протокол обнаружения канального уровня может использоваться как компонент в приложениях [управления сетью](https://ru.wikibrief.org/wiki/Network_management) и [мониторинга сети](https://ru.wikibrief.org/wiki/Network_monitoring).

Одним из таких примеров является его использование в [требованиях мостового соединения центра обработки данных](https://ru.wikibrief.org/wiki/Data_center_bridging). Протокол обмена возможностями моста центра обработки данных (DCBX) - это протокол обнаружения и обмена возможностями, который используется для передачи возможностей и настройки вышеуказанных функций между соседями для обеспечения согласованной конфигурации в сети.

LLDP используется для объявления [возможностей и требований питания через Ethernet](https://ru.wikibrief.org/wiki/Power_over_Ethernet), а также для согласования подачи питания.