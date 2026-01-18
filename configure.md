
## 📊 Топология сети
                      [Router]
                         |
                  Gig0/0.10-99 (Subinterfaces)
                         |
                     [Core-SW] (SW1)
                    /           \
          Port-channel1        Port-channel2
               /                     \
        [Floor1-SW] (SW2)       [Floor2-SW] (SW3)
          /        \               /        \
       [PC1]     [PC2]         [PC3]     [Server]
     VLAN 20    VLAN 30       VLAN 40    VLAN 50
     Sales     Engineering    Finance    Servers


**Физическая схема:**
- Маршрутизатор Cisco 2911
- 3 коммутатора Cisco 2960
- 4 конечных устройства (3 ПК + 1 сервер)

## 🎯 Цели лабораторной работы

### Основные цели:
-  Настроить VLAN для разных отделов
-  Реализовать EtherChannel между коммутаторами
-  Настроить маршрутизацию между VLAN

### задачи:
1. Создать и распределить VLAN
2. Настроить транковые порты между коммутаторами
3. Создать EtherChannel для агрегации каналов
4. Настроить маршрутизатор по схеме

## 🔧 Конфигурация VLAN

| VLAN ID | Имя         | Подсеть          |  `Gateway`  | `Назначение `              |
|---------|-------------|------------------|-------------|----------------------------|
| 2       | sales       | 192.168.2.0/24   | 192.168.2.1 | Управление коммутаторами   |
| 3       | Engineering | 192.168.3.0/24   | 192.168.3.1 | Инженерный отдел           |
| 4       | Finance     | 192.168.4.0/24   | 192.168.4.1 | Финансовый отдел           |
| 5       | Servers     | 192.168.5.0/24   | 192.168.5.1 | Серверная                  |

## ⚡ Настройка EtherChannel

### Параметры EtherChannel:
- **Тип:** LACP (Link Aggregation Control Protocol)
- **Режим:** Active-Active
- **Количество линков:** 2 на каждый канал
- **Номер группы:** Port-channel 1, Port-channel 2

### Схема EtherChannel соединений:
### Конфигурация на  (SW0):

switch0> en <br>
switch0# conf t <br>
switch0(config)#vlan 2  **создаем vlan** <br>
switch0(config-vlan)# name sales  **название vlan** <br>
switch0(config-vlan)# ex <br>
switch0(config)#vlan 3  <br>
switch0(config-vlan)# name Engineering   <br>
switch0(config-vlan)# ex <br>
switch0(config)#vlan 4  <br>
switch0(config-vlan)# name Finance <br>  
switch0(config-vlan)# ex <br>
switch0(config)#vlan 5 <br>
switch0(config-vlan)# name `Servers` <br>  
switch0(config-vlan)# ex <br>
switch0(config)# int g0/1  **переходим в режим интерфейса**  <br>
switch0(config-if)# switchport mode trunk  **обозначаем интерфейс как trunk , это озночает что по нему теперь могут ходить тегированные frame**<br>
switch0(config-if)# switchport trunk all vlan 2,3,4,5  **назначем номера vlan которые может пропускать этот интерфейс** <br> 
switch0(config-if)# ex <br>
switch0(config)# `int range f0/1-2` **переходим в режим настройки нескольких интерфейсов одновременно** <br>
switch0(config-if-range)# `channel-group 1 mode active` **создаем объединенный интерфейс** <br>
switch0(config-if-range)# ex <br>
switch0(config)# `int Port-channel  1` **переходим в режим объединенного интерфейса**  <br> 

switch0(config-if)# switchport mode trunk <br>
switch0(config-if)# switchport trunk all vlan 2,3,4,5 <br>
switch0(config-if)# ex <br>
switch0(config)# int range f0/3-4 <br>
switch0(config-if-range)# channel-group 2 mode active <br>
switch0(config-if-range)# ex <br>
switch0(config)# int Port-channel  2 <br>
switch0(config-if)# switchport mode trunk <br>
switch0(config-if)# switchport trunk all vlan 2,3,4,5 <br>
switch0(config-if)# ex <br>

### Конфигурация на  (SW1): <br>
switch1> en <br>
switch1# conf t <br>
switch1(config)#vlan 2  <br>
switch1(config-vlan)# name sales  <br>
switch1(config-vlan)# ex <br>
switch1(config)#vlan 3  <br>
switch1(config-vlan)# name Engineering   <br>
switch1(config-vlan)# ex <br>
switch1(config)#vlan 4  <br>
switch1(config-vlan)# name Finance   <br>
switch1(config-vlan)# ex <br>
switch1(config)#vlan 5 <br>
switch1(config-vlan)# name Servers   <br>
switch1(config-vlan)# ex  <br>
switch1(config)# int range f0/1-2 <br>
switch1(config-if-range)# channel-group 1 mode active <br>
switch1(config-if-range)# ex <br>
switch1(config)# int Port-channel  1  <br>
switch1(config-if)# switchport mode trunk <br>
switch1(config-if)# switchport trunk all vlan 2,3,4,5 <br>
switch1(config-if)# ex <br>
switch1(config)# int  f0/3 <br>
switch1(config-if)# `switchport mode access` **указываем что данный интерфейс является интерфейсом доступа** <br>
switch1(config-if)# `switchport access vlan 2` **указываем vlan этого интерфейса** <br>
switch1(config)# int  f0/4 <br>
switch1(config-if)# `switchport mode access` <br>
switch1(config-if)# `switchport access vlan3` <br>

### Конфигурация на  (SW2): <br>
switch2> en  <br>
switch2# conf t <br>
switch2(config)#vlan 2   <br>
switch2(config-vlan)# name sales  <br>
switch2(config-vlan)# ex <br>
switch2(config)#vlan 3  <br>
switch2(config-vlan)# name Engineering   <br>
switch2(config-vlan)# ex <br>
switch2(config)#vlan 4  <br>
switch2(config-vlan)# name Finance   <br>
switch2(config-vlan)# ex <br>
switch2(config)#vlan 5 <br>
switch2(config-vlan)# name Servers   <br>
switch2(config-vlan)# ex  <br>
switch2(config)# int range f0/3-4 <br>
switch2(config-if-range)# channel-group 2 mode active  <br>
switch2(config-if-range)# ex <br>
switch2(config)# int Port-channel  2  <br>
switch2(config-if)# switchport mode trunk <br>
switch2(config-if)# switchport trunk all vlan 2,3,4,5 <br>
switch2(config-if)# ex <br>
switch2(config)# int  f0/1 <br>
switch2(config-if)# switchport mode access <br>
switch2(config-if)# switchport access vlan 3 <br>
switch2(config)# int  f0/4 <br>
switch2(config-if)# switchport mode access <br>
switch2(config-if)# switchport access vlan 4 <br>


### Конфигурация на Router0 <br>
router> en <br>
router# conf t <br>
router(config)# int g0/0  **переходим в режим интерфейса** <br>
router(config-if)# no shutdown  **физически включаем** <br>
router(config-if)# ex   <br>
router(config)# int g 0/0.2   **на физическом интерфейсе создаем фиртуальный для vlan** <br>
router(config-subif)# encapsulation dot1Q 2 **эта команда тегирует трафик vlan, добавляет свой заголовок во frame** <br>
router(config-subif)# ip addr 192.168.2.1 255.255.255.0  **задаём ip адресс шлюза по умолчаннию и маску сети** <br>
router(config-subif)# ex <br>
router(config)# int g 0/0.3 <br>
router(config-subif)# `encapsulation` dot1Q 3 <br>
router(config-subif)# ip addr 192.168.3.1 255.255.255.0  <br>
router(config-subif)# ex <br>
router(config)# int g 0/0.4 <br>
router(config-subif)# encapsulation dot1Q 4 <br>
router(config-subif)# ip addr 192.168.4.1 255.255.255.0  <br>
router(config-subif)# ex <br>
router(config)# int g 0/0.5 <br>
router(config-subif)# encapsulation dot1Q 5 <br>
router(config-subif)# ip addr 192.168.5.1 255.255.255.0  <br>
router(config-subif)# ex <br>

















