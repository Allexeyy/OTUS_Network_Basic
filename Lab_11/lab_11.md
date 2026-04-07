**_Лабораторная работа №11._**

*Настройка и проверка расширенных списков контроля доступа*

ТОПОЛОГИЯ

![](Lab_11.jpg)
![](Lab_11_0.jpg)
![](Lab_11_1.jpg)

# Цели
    Часть 1. Создание сети и настройка основных параметров устройства
    Часть 2. Настройка сетей VLAN на коммутаторах.
    Часть 3. Настройте транки (магистральные каналы).
    Часть 4. Настройте маршрутизацию.
    Часть 5. Настройте удаленный доступ
    Часть 6. Проверка подключения
    Часть 7. Настройка и проверка списков контроля доступа (ACL)


ВАЖНО: Native VLAN 666
-----------------------------------------------------

# Часть 1. Создание сети и настройка основных параметров устройства

1.1 - 1.3 Создали и настройка сети согласно топологии
Базовая настройка роутера и коммутаторов на основве файла настроек.

Меняем при внесении конфигурации только имя хоста оборудования согласно схемы: R!, R2, S1, S2

    !
    service password-encryption
    !
    hostname S1
    !
    enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
    !
    banner motd ^C
    *******************************************************
    ****     Caution! Enter only adminstrator OTUS     ****
    *******************************************************^C
    !
    line con 0
    password 7 0822455D0A16
    logging synchronous
    login
    !
    line aux 0
    login
    !
    line vty 0 4
    password 7 0822455D0A16
    logging synchronous
    login
    line vty 5 15
    password 7 0822455D0A16
    logging synchronous
    login
    !
    end

 и сохраняем конфигурацию

    copy running-config startup-config 
    или
    write memory

# Часть 2. ННастройка сетей VLAN на коммутаторах.

2.1 - 2.2 Создание и настройка VLAN согласно задачи

    S1#show vlan brief 

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    
    20   Managment                        active    
    30   Operations                       active    Fa0/6
    40   Sales                            active    
    666  NATIVE                           active    
    999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/7
                                                    Fa0/8, Fa0/9, Fa0/10, Fa0/11
                                                    Fa0/12, Fa0/13, Fa0/14, Fa0/15
                                                    Fa0/16, Fa0/17, Fa0/18, Fa0/19
                                                    Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                    Fa0/24, Gig0/1, Gig0/2
    1000 MySelf                           active    
    1002 fddi-default                     active    
    1003 token-ring-default               active    
    1004 fddinet-default                  active    
    1005 trnet-default                    active    
    S1#

    S2#show vlan brief 

    VLAN Name                             Status    Ports
    ---- -------------------------------- --------- -------------------------------
    1    default                          active    
    20   Managment                        active    Fa0/5
    30   Operations                       active    
    40   Sales                            active    Fa0/18
    666  NATIVE                           active    
    999  ParkingLot                       active    Fa0/2, Fa0/3, Fa0/4, Fa0/6
                                                    Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                    Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                    Fa0/15, Fa0/16, Fa0/17, Fa0/19
                                                    Fa0/20, Fa0/21, Fa0/22, Fa0/23
                                                    Fa0/24, Gig0/1, Gig0/2
    1000 MySelf                           active    
    1002 fddi-default                     active    
    1003 token-ring-default               active    
    1004 fddinet-default                  active    
    1005 trnet-default                    active    
    S2#


# Часть 3. Настройте транки (магистральные каналы)

3.1  Реализация различных оптимизаций на каждом маршрутизаторе

        R1(config)#interface g 0/0/1
        R1(config-if)#ip ospf priority 50
            БЫЛО
            R1#sh ip ospf neighbor 
            Neighbor ID     Pri   State           Dead Time   Address         Interface
            2.2.2.2           1   FULL/DR         00:01:41    10.53.0.2       GigabitEthernet0/0/1

            R2#sh ip os ne
            Neighbor ID     Pri   State           Dead Time   Address         Interface
            1.1.1.1           1   FULL/BDR        00:01:58    10.53.0.1       GigabitEthernet0/0/1

            СТАЛО    
            R1#sh ip ospf neighbor 
            Neighbor ID     Pri   State           Dead Time   Address         Interface
            2.2.2.2           1   FULL/BDR        00:01:41    10.53.0.2       GigabitEthernet0/0/1

            R2#sh ip os ne
            Neighbor ID     Pri   State           Dead Time   Address         Interface
            1.1.1.1          50   FULL/DR         00:01:58    10.53.0.1       GigabitEthernet0/0/1


        R1(config-if)#ip ospf hello-interval 30
        R1(config-if)#ip ospf dead-interval 120

        R1(config)#ip route 0.0.0.0 0.0.0.0 loopback 1
        %Default route without gateway, if not a point-to-point interface, may impact performance
        R1(config)#router ospf 56
        R1(config-router)#default-information originate 

    R2(config)#interface loopback 1
    R2(config-if)#ip ospf network point-to-point 
    
    R1#sh ip route ospf 
        БЫЛО
        192.168.1.0/32 is subnetted, 1 subnets
    O       192.168.1.1 [110/2] via 10.53.0.2, 00:10:02, GigabitEthernet0/0/1
    
        СТАЛО
    O    192.168.1.0 [110/2] via 10.53.0.2, 00:01:13, GigabitEthernet0/0/1

            R2(config)#router ospf 56
            R2(config-router)#passive-interface loopback 1   

            R1#sh ip ospf interface g 0/0/1
            БЫЛО
            GigabitEthernet0/0/1 is up, line protocol is up
            Internet address is 10.53.0.1/24, Area 0
            Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 1

            СТАЛО
            GigabitEthernet0/0/1 is up, line protocol is up
            Internet address is 10.53.0.1/24, Area 0
            Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 35     



3.2. Убедитесь, что оптимизация OSPFv2 реализовалась.

    R1#show ip ospf interface g0/0/1 

    GigabitEthernet0/0/1 is up, line protocol is up
    Internet address is 10.53.0.1/24, Area 0
    Process ID 56, Router ID 1.1.1.1, Network Type BROADCAST, Cost: 35
    Transmit Delay is 1 sec, State DR, Priority 50
    Designated Router (ID) 1.1.1.1, Interface address 10.53.0.1
    Backup Designated Router (ID) 2.2.2.2, Interface address 10.53.0.2
    Timer intervals configured, Hello 30, Dead 120, Wait 120, Retransmit 5
        Hello due in 00:00:09
    Index 1/1, flood queue length 0
    Next 0x0(0)/0x0(0)
    Last flood scan length is 1, maximum is 1
    Last flood scan time is 0 msec, maximum is 0 msec
    Neighbor Count is 1, Adjacent neighbor count is 1
        Adjacent with neighbor 2.2.2.2  (Backup Designated Router)
    Suppress hello for 0 neighbor(s)


            R2#ping 172.16.1.1

            Type escape sequence to abort.
            Sending 5, 100-byte ICMP Echos to 172.16.1.1, timeout is 2 seconds:
            !!!!!
            Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/1 ms


    R2#sh ip route ospf 
    O*E2 0.0.0.0/0 [110/1] via 10.53.0.1, 00:00:24, GigabitEthernet0/0/1

    R1#sh ip route ospf 
    O    192.168.1.0 [110/36] via 10.53.0.2, 00:00:09, GigabitEthernet0/0/1


Вопросы для повторения:

    1.	Какой маршрутизатор является DR? Какой маршрутизатор является BDR? Каковы критерии отбора?
    Ответ: Изначально при равных приоритетах R1 - BDR, а R2 - DR т.к. ID 2го роутера больше 1го. Когда в п.3.1 согласно задания повысили приоритет R1 до 50, после перезагрузки роутера или очистки процессора их функции поменялись местами (если очистку процесса или перезагрузку не делать, то изменений не будет)


    2.	Почему стоимость OSPF для маршрута по умолчанию отличается от стоимости OSPF в R1 для сети 192.168.1.0/24?
    Ответ: Т.к. стоимость пути рассчитывается как сумма затрат на подключение по всему пути то разность стоимостей будет различаться на 1 т.к. у нас есть стоимость пути внутри маршрутизатора между внешним интерфейсом и внутренним loopback равным 1

   
Файл схемы сети [здесь](Lab_10/lab_10.pkt).

- [Вернуться на основную страницу ](/readme.md)



