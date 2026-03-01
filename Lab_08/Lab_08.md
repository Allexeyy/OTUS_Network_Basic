**_Лабораторная работа №08._**

*Реализация DHCPv4*

ТОПОЛОГИЯ

![](/Lab_08/Lab_08.jpg)

![](/Lab_08/Lab_08_1.jpg)

# Задачи
 
    Часть 1. Создание сети и настройка основных параметров устройства
    Часть 2. Настройка и проверка двух серверов DHCPv4 на R1
    Часть 3. Настройка и проверка DHCP-ретрансляции на R2


-----------------------------------------------------

# Часть 1. Создание сети и настройка основных параметров устройства

1.1 - 1.9 Создали сеть согласно топологии
Базовая настройка роутера и коммутаторов на основве файла настроек.
Рекомендуется делать на окончательном этапе, чтобы часто не вводить пароли
Поменять: 

    hostname & ip domain-name 
соответствующие тому оборудованию на котором будет накатываться конфигурация

перед копированием и вставкой
Зайти в режим глобальной конфигурации

(config)# 

    service password-encryption
    !
    hostname S1
    !
    enable secret 5 $1$mERr$9cTjUIEqNGurQiFU.ZeCi1
    !
    no ip domain-lookup
    ip domain-name SW1
    !
    username admin secret 5 $1$mERr$hx5rVt7rPNoS4wqbXKX7m0
    !
    banner motd # Unauthorized access is prohibited! #
    !
    line con 0
     password 7 0822455D0A16
     login
     exec-timeout 5 0
     logging synchronous
    !
    line vty 0 4
     exec-timeout 5 0
     password 7 0822455D0A16
     login local
     transport input ssh
    line vty 5 15
     exec-timeout 5 0
     password 7 0822455D0A16
     login local
     transport input ssh
    !
    end

Выполнить комманды

    (config)# crypto key generate rsa general-keys modulus 1024
    (config)# ip ssh version 2
    #w
    #reload
Генерация ssh-ключа на основе локальных данных конфигурации, переключение на новую версию ssh, сохранение и перезагрузка обррудования.

Настройка коммутаторов согласно задачи

    Switch(config)#hostname S1
    S1(config)#no ip domain-name
    S1(config)#vlan 100
    S1(config-vlan)#name CLIENT
    S1(config-vlan)#exit
    S1(config)#vlan 200
    S1(config-vlan)#name MANAGMENT
    S1(config-vlan)#exit
    S1(config)#vlan 999
    S1(config-vlan)#name ParkingLot
    S1(config-vlan)#exit
    S1(config)#vlan 1000
    S1(config-vlan)#name NATIVE
    S1(config-vlan)#exit
    S1(config)#int f0/6
    S1(config-if)#switchport mode access 
    S1(config-if)#switchport access vlan 100
    S1(config-if)#no sh
    S1(config)#int r f0/1-4,f0/7-24,g0/1-2
    S1(config-if-range)#switchport mode access 
    S1(config-if-range)#switchport access vlan 999
    S1(config-if-range)#sh
    S1(config)#interface f0/5
    S1(config-if)#switchport mode trunk 
    S1(config-if)#switchport trunk native vlan 1000
    S1(config-if)#switchport trunk allowed vlan 100,200,1000
    S1(config-if)#switchport nonegotiate 
    S1(config-if)#no sh
    S1(config)#ip default-gateway 192.168.1.68

        Router(config)#hostname R1
        R1(config)#no ip domain-name
        R1(config)#interface g0/0/1.100
        R1(config-subif)#encapsulation dot1Q 100
        R1(config-subif)#ip address 192.168.1.1 255.255.255.192
        R1(config-subif)#exit
        R1(config)#interface g0/0/1.200
        R1(config-subif)#encapsulation dot1Q 200
        R1(config-subif)#ip address 192.168.1.68 255.255.255.224
        R1(config-subif)#exit
        R1(config)#int g0/0/1.1000
        R1(config-subif)#encapsulation dot1Q 1000 native 
        R1(config-subif)#exit
        R1(config)#int g0/0/0
        R1(config-if)#ip address 10.0.0.1 255.255.255.252
        R1(config-if)#no sh
        R1(config)#ip route 0.0.0.0 0.0.0.0 10.0.0.2

--------
    Вопрос: Почему интерфейс F0/5 указан в VLAN 1?
    Ответ:  На момент настройки данный интерфейс не прописан как TRUNC.

    Вопрос: Какой IP-адрес был бы у ПК, если бы он был подключен к сети с помощью DHCP?
    Ответ:  Все зависит от настроек самого DHCP-сервера, но исходя из задания адрес присваивается из начала исходного промежутка адресов т.е. 192.168.1.6

# Часть 2.	Настройка и проверка двух серверов DHCPv4 на R1

2.1 - 2.2. Настройте R1 с пулами DHCPv4 для двух поддерживаемых подсетей. Ниже приведен только пул DHCP для подсети A

    ip dhcp excluded-address 192.168.1.1 192.168.1.5
    ip dhcp excluded-address 192.168.1.97 192.168.1.99
    ip dhcp pool CCNA-lab.com
        network 192.168.1.0 255.255.255.192
        default-router 192.168.1.1
        dns-server 192.168.1.5
        domain-name CCNA-lab.com

    ip dhcp pool R2_Client_LAN
        network 192.168.1.96 255.255.255.240
        default-router 192.168.1.97
        dns-server 192.168.1.5
        domain-name CCNA-lab.com

ареннда IP адреса (lease 2 12 30)   - нельзя выполнить в текущей версии программы

2.3. Проверка конфигурации сервера DHCPv4
![](/Lab_08/Lab_08_2.jpg)
![](/Lab_08/Lab_08_3.jpg)
show ip dhcp server statistics  - нельзя выполнить в текущей версии программы

2.4. Попытка получить IP-адрес от DHCP на PC-A
![](/Lab_08/Lab_08_4.jpg)
![](/Lab_08/Lab_08_5.jpg)
![](/Lab_08/Lab_08_6.jpg)

# Часть 3.	Настройка и проверка DHCP-ретрансляции на R2

3.1. Настройка R2 в качестве агента DHCP-ретрансляции для локальной сети на G0/0/1

    !
    interface GigabitEthernet0/0/1.100
    encapsulation dot1Q 100
    ip address 192.168.1.97 255.255.255.240
    ip helper-address 10.0.0.1
    !

3.2. Попытка получить IP-адрес от DHCP на PC-B
![](/Lab_08/Lab_08_7.jpg)
![](/Lab_08/Lab_08_8.jpg)
![](/Lab_08/Lab_08_9.jpg)
![](/Lab_08/Lab_08_10.jpg)

Файл схемы сети [здесь](Lab_08/lab_08.pkt).

- [Вернуться на основную страницу ](..readme.md)



