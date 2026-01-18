

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

| VLAN ID | Имя         | Подсеть          |  Шлюз       | Назначение                 |
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
switch0> en 
switch0# conf t
switch0(config)#vlan 2  **создаем vlan**
switch0(config-vlan)# name sales  **название vlan** 
switch0(config-vlan)# ex 
switch0(config)#vlan 3  
switch0(config-vlan)# name Engineering   
switch0(config-vlan)# ex 
switch0(config)#vlan 4  
switch0(config-vlan)# name Finance   
switch0(config-vlan)# ex 
switch0(config)#vlan 5 
switch0(config-vlan)# name Servers   
switch0(config-vlan)# ex 
switch0(config)# int g0/1  **переходим в режим интерфейса**
switch0(config-if)# switchport mode trunk  **обозначаем интерфейс как trunk , это озночает что по нему теперь могут ходить тегированные frame**
switch0(config-if)# switchport trunk all vlan 2,3,4,5  **назначем номера vlan которые может пропускать этот интерфейс** 
switch0(config-if)# ex 
switch0(config)# int range f0/1-2 **переходим в режим настройки нескольких интерфейсов одновременно** 
switch0(config-if-range)# channel-group 1 mode active  **создаем объединенный интерфейс** 
switch0(config-if-range)# ex 
switch0(config)# int Port-channel  1  **переходим в режим объединенного интерфейса** <br>
switch0(config-if)# switchport mode trunk 
switch0(config-if)# switchport trunk all vlan 2,3,4,5 
switch0(config-if)# ex 
switch0(config)# int range f0/3-4 
switch0(config-if-range)# channel-group 2 mode active 
switch0(config-if-range)# ex 
switch0(config)# int Port-channel  2
switch0(config-if)# switchport mode trunk 
switch0(config-if)# switchport trunk all vlan 2,3,4,5 
switch0(config-if)# ex 

### Конфигурация на  (SW1):
switch1> en 
switch1# conf t
switch1(config)#vlan 2  
switch1(config-vlan)# name sales  
switch1(config-vlan)# ex 
switch1(config)#vlan 3  
switch1(config-vlan)# name Engineering   
switch1(config-vlan)# ex 
switch1(config)#vlan 4  
switch1(config-vlan)# name Finance   
switch1(config-vlan)# ex 
switch1(config)#vlan 5 
switch1(config-vlan)# name Servers   
switch1(config-vlan)# ex  
switch1(config)# int range f0/1-2
switch1(config-if-range)# channel-group 1 mode active 
switch1(config-if-range)# ex 
switch1(config)# int Port-channel  1  
switch1(config-if)# switchport mode trunk 
switch1(config-if)# switchport trunk all vlan 2,3,4,5 
switch1(config-if)# ex 
switch1(config)# int  f0/3
switch1(config-if)# switchport mode access **указываем что данный интерфейс является интерфейсом доступа**
switch1(config-if)# switchport access vlan 2 **указываем vlan этого интерфейса**
switch1(config)# int  f0/4
switch1(config-if)# switchport mode access 
switch1(config-if)# switchport access vlan3

### Конфигурация на  (SW2):
switch2> en 
switch2# conf t
switch2(config)#vlan 2  
switch2(config-vlan)# name sales  
switch2(config-vlan)# ex 
switch2(config)#vlan 3  
switch2(config-vlan)# name Engineering   
switch2(config-vlan)# ex 
switch2(config)#vlan 4  
switch2(config-vlan)# name Finance   
switch2(config-vlan)# ex 
switch2(config)#vlan 5 
switch2(config-vlan)# name Servers   
switch2(config-vlan)# ex  
switch2(config)# int range f0/3-4
switch2(config-if-range)# channel-group 2 mode active 
switch2(config-if-range)# ex 
switch2(config)# int Port-channel  2  
switch2(config-if)# switchport mode trunk 
switch2(config-if)# switchport trunk all vlan 2,3,4,5 
switch2(config-if)# ex 
switch2(config)# int  f0/1
switch2(config-if)# switchport mode access 
switch2(config-if)# switchport access vlan 3 
switch2(config)# int  f0/4
switch2(config-if)# switchport mode access 
switch2(config-if)# switchport access vlan 4


### Конфигурация на Router0
router> en 
router# conf t 
router(config)# int g0/0  **переходим в режим интерфейса** 
router(config-if)# no shutdown  **физически включаем** 
router(config-if)# ex   
router(config)# int g 0/0.2   **на физическом интерфейсе создаем фиртуальный для vlan** 
router(config-subif)# encapsulation dot1Q 2 **эта команда тегирует трафик vlan, добавляет свой заголовок во frame** 
router(config-subif)# ip addr 192.168.2.1 255.255.255.0  **задаём ip адресс шлюза по умолчаннию и маску сети** 
router(config-subif)# ex 
router(config)# int g 0/0.3
router(config-subif)# encapsulation dot1Q 3 
router(config-subif)# ip addr 192.168.3.1 255.255.255.0  
router(config-subif)# ex 
router(config)# int g 0/0.4
router(config-subif)# encapsulation dot1Q 4 
router(config-subif)# ip addr 192.168.4.1 255.255.255.0  
router(config-subif)# ex 
router(config)# int g 0/0.5
router(config-subif)# encapsulation dot1Q 5 
router(config-subif)# ip addr 192.168.5.1 255.255.255.0  
router(config-subif)# ex

 



