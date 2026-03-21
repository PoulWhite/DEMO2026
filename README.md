# DEMO2026
================================================================================
        МОДУЛЬ 1: ПОЛНАЯ ПОШАГОВАЯ ИНСТРУКЦИЯ (КОД 09.02.06-1-2026)
        Файл: ФамилияУчастникаМодуль1.txt
        Версия: 1.0 (подробная шпаргалка)
================================================================================

СОДЕРЖАНИЕ:
1.  Вводная информация
2.  Тайм-менеджмент
3.  Шаг 1: Базовая настройка устройств (имена, IP)
4.  Шаг 2: Настройка VLAN на HQ-RTR
5.  Шаг 3: Создание локальных учетных записей
6.  Шаг 4: Настройка ISP (доступ в интернет)
7.  Шаг 5: Настройка безопасного SSH на серверах
8.  Шаг 6: Настройка IP-туннеля (GRE)
9.  Шаг 7: Динамическая маршрутизация (OSPF) с паролем
10. Шаг 8: NAT на HQ-RTR и BR-RTR
11. Шаг 9: DHCP для HQ-CLI
12. Шаг 10: DNS-сервер на HQ-SRV
13. Шаг 11: Настройка часового пояса
14. Шаг 12: Оформление отчета
15. Финальный чек-лист

================================================================================
1. ВВОДНАЯ ИНФОРМАЦИЯ
================================================================================
Время выполнения: 1 час (60 минут)
Что нужно сдать: рабочая сеть + отчет
Имя файла отчета: ФамилияУчастникаМодуль1.docx или .pdf

Топология:
- Офис HQ: HQ-RTR, HQ-SRV, HQ-CLI (VLAN 100, 200, 999)
- Офис BR: BR-RTR, BR-SRV
- ISP: соединяет HQ-RTR и BR-RTR с интернетом

Дистрибутивы:
- HQ-SRV, BR-SRV, HQ-CLI: Альт Сервер / Альт Рабочая станция
- HQ-RTR, BR-RTR: EcoRouter (или Альт JeOS с FRR)
- ISP: Альт JeOS

================================================================================
2. ТАЙМ-МЕНЕДЖМЕНТ (60 МИНУТ)
================================================================================
| Этап | Действие                                     | Время  |
|------|----------------------------------------------|--------|
| 1    | Базовая настройка (имена, IP-адреса)         | 10 мин |
| 2    | VLAN на HQ-RTR                               | 5 мин  |
| 3    | Пользователи (sshuser, net_admin)            | 5 мин  |
| 4    | ISP (DHCP, NAT)                              | 5 мин  |
| 5    | SSH на серверах (порт 2026, баннер)          | 5 мин  |
| 6    | IP-туннель (GRE)                             | 5 мин  |
| 7    | OSPF с паролем                               | 5 мин  |
| 8    | NAT на HQ-RTR и BR-RTR                       | 5 мин  |
| 9    | DHCP для HQ-CLI                              | 5 мин  |
| 10   | DNS-сервер на HQ-SRV                         | 10 мин |
| 11   | Часовой пояс                                 | 2 мин  |
| 12   | Отчет (скриншоты, таблицы)                   | 8 мин  |

================================================================================
3. ШАГ 1: БАЗОВАЯ НАСТРОЙКА УСТРОЙСТВ
================================================================================

3.1 Настройка имени хоста (FQDN)
--------------------------------
На КАЖДОМ устройстве выполняем команду:

hostnamectl set-hostname <имя>.au-team.irpo

Примеры:
- HQ-RTR:   hostnamectl set-hostname hq-rtr.au-team.irpo
- BR-RTR:   hostnamectl set-hostname br-rtr.au-team.irpo
- HQ-SRV:   hostnamectl set-hostname hq-srv.au-team.irpo
- BR-SRV:   hostnamectl set-hostname br-srv.au-team.irpo
- HQ-CLI:   hostnamectl set-hostname hq-cli.au-team.irpo
- ISP:      hostnamectl set-hostname isp

Проверка: hostnamectl

3.2 Расчет IP-адресации (VLSM)
------------------------------
Требования задания:
- HQ-SRV (VLAN 100): ≤ 32 адреса -> маска /27 (32 адреса)
- HQ-CLI (VLAN 200): ≥ 16 адресов -> маска /28 (16 адресов)
- Управление (VLAN 999): ≤ 8 адресов -> маска /29 (8 адресов)
- BR-SRV: ≤ 16 адресов -> маска /28

ВЫБРАННЫЕ СЕТИ (из RFC 1918):

| VLAN/сеть      | Сеть           | Маска | Диапазон               | Шлюз            |
|----------------|----------------|-------|------------------------|-----------------|
| VLAN 100       | 192.168.10.0   | /27   | 192.168.10.1-30        | 192.168.10.1    |
| VLAN 200       | 192.168.10.32  | /28   | 192.168.10.33-46       | 192.168.10.33   |
| VLAN 999       | 192.168.10.48  | /29   | 192.168.10.49-54       | 192.168.10.49   |
| BR-SRV         | 192.168.20.0   | /28   | 192.168.20.1-14        | 192.168.20.1    |

3.3 Таблица адресов (заполнить в отчете)
----------------------------------------
| Имя устройства | IP-адрес                                    | Шлюз по умолчанию |
|----------------|---------------------------------------------|-------------------|
| HQ-RTR         | 192.168.10.1/27, 192.168.10.33/28, 192.168.10.49/29 | —                 |
| BR-RTR         | 192.168.20.1/28                             | —                 |
| HQ-SRV         | 192.168.10.2/27                             | 192.168.10.1      |
| BR-SRV         | 192.168.20.2/28                             | 192.168.20.1      |
| HQ-CLI         | 192.168.10.34/28                            | 192.168.10.33     |

3.4 Назначение IP-адресов (временные, до настройки VLAN)
--------------------------------------------------------
На HQ-RTR (на физический интерфейс eth0, позже заменим на VLAN):
  ip addr add 192.168.10.1/27 dev eth0
  ip addr add 192.168.10.33/28 dev eth0
  ip addr add 192.168.10.49/29 dev eth0

На BR-RTR:
  ip addr add 192.168.20.1/28 dev eth0

На HQ-SRV:
  ip addr add 192.168.10.2/27 dev eth0
  ip route add default via 192.168.10.1

На BR-SRV:
  ip addr add 192.168.20.2/28 dev eth0
  ip route add default via 192.168.20.1

На HQ-CLI:
  ip addr add 192.168.10.34/28 dev eth0
  ip route add default via 192.168.10.33

================================================================================
4. ШАГ 2: НАСТРОЙКА VLAN НА HQ-RTR (С ИСПОЛЬЗОВАНИЕМ ОДНОГО ФИЗИЧЕСКОГО ПОРТА)
================================================================================

На HQ-RTR (предполагаем, что физический интерфейс называется eth0):

# Создаем VLAN-интерфейсы
ip link add link eth0 name eth0.100 type vlan id 100
ip link add link eth0 name eth0.200 type vlan id 200
ip link add link eth0 name eth0.999 type vlan id 999

# Включаем их
ip link set eth0.100 up
ip link set eth0.200 up
ip link set eth0.999 up

# Назначаем IP-адреса на VLAN-интерфейсы
ip addr add 192.168.10.1/27 dev eth0.100
ip addr add 192.168.10.33/28 dev eth0.200
ip addr add 192.168.10.49/29 dev eth0.999

# Удаляем старые IP с физического интерфейса (если были)
ip addr del 192.168.10.1/27 dev eth0
ip addr del 192.168.10.33/28 dev eth0
ip addr del 192.168.10.49/29 dev eth0

# Включаем IP-форвардинг (для маршрутизации между VLAN)
echo 1 > /proc/sys/net/ipv4/ip_forward

================================================================================
5. ШАГ 3: СОЗДАНИЕ ЛОКАЛЬНЫХ УЧЕТНЫХ ЗАПИСЕЙ
================================================================================

5.1 На HQ-SRV и BR-SRV (серверы)
--------------------------------
# Создаем пользователя sshuser с UID 2026
useradd -m -u 2026 sshuser
echo "sshuser:P@ssw0rd" | chpasswd

# Даем право sudo без пароля
echo "sshuser ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/sshuser
chmod 440 /etc/sudoers.d/sshuser

5.2 На HQ-RTR и BR-RTR (маршрутизаторы)
---------------------------------------
# Создаем пользователя net_admin
useradd -m net_admin
echo "net_admin:P@ssw0rd" | chpasswd

# Для EcoRouter - добавляем в группу администраторов
usermod -aG wheel net_admin

# Для Linux (Альт) - даем sudo без пароля
echo "net_admin ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/net_admin
chmod 440 /etc/sudoers.d/net_admin

================================================================================
6. ШАГ 4: НАСТРОЙКА ISP (ДОСТУП В ИНТЕРНЕТ)
================================================================================

ISP имеет:
- eth0 -> в магистрального провайдера (получает адрес по DHCP)
- eth1 -> к HQ-RTR (сеть 172.16.1.0/28)
- eth2 -> к BR-RTR (сеть 172.16.2.0/28)

Выполняем на ISP:

# 1. Интерфейс к провайдеру - получаем адрес по DHCP
dhclient eth0

# 2. Интерфейсы к офисам
ip addr add 172.16.1.1/28 dev eth1
ip addr add 172.16.2.1/28 dev eth2
ip link set eth1 up
ip link set eth2 up

# 3. Маршрут по умолчанию (из DHCP)
#    Получаем gateway из вывода: ip route show
#    Добавляем:
ip route add default via <gateway_полученный_от_dhcp>

# 4. Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# 5. Настраиваем NAT (динамическая трансляция портов)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 6. Разрешаем форвардинг между интерфейсами
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth2 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT

# 7. Сохраняем правила (для Альт Linux)
iptables-save > /etc/sysconfig/iptables

================================================================================
7. ШАГ 5: НАСТРОЙКА БЕЗОПАСНОГО SSH НА HQ-SRV И BR-SRV
================================================================================

Выполняем НА КАЖДОМ СЕРВЕРЕ (HQ-SRV и BR-SRV):

# 1. Редактируем конфиг SSH
nano /etc/ssh/sshd_config

# Вносим изменения:
Port 2026
PermitRootLogin no
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh/banner.txt

# 2. Создаем баннер
echo "Authorized access only" > /etc/ssh/banner.txt

# 3. Перезапускаем SSH
systemctl restart sshd

# 4. Проверяем, что слушается порт 2026
ss -tlnp | grep 2026

================================================================================
8. ШАГ 6: НАСТРОЙКА IP-ТУННЕЛЯ (GRE)
================================================================================

8.1 На HQ-RTR
-------------
# IP-адреса для туннеля
# local = адрес ISP-интерфейса HQ-RTR (172.16.1.2)
# remote = адрес ISP-интерфейса BR-RTR (172.16.2.2)

ip tunnel add tun0 mode gre remote 172.16.2.2 local 172.16.1.2
ip addr add 10.0.0.1/30 dev tun0
ip link set tun0 up

8.2 На BR-RTR
-------------
ip tunnel add tun0 mode gre remote 172.16.1.2 local 172.16.2.2
ip addr add 10.0.0.2/30 dev tun0
ip link set tun0 up

8.3 Проверка
------------
# С HQ-RTR пингуем BR-RTR через туннель
ping 10.0.0.2

================================================================================
9. ШАГ 7: ДИНАМИЧЕСКАЯ МАРШРУТИЗАЦИЯ (OSPF) С ПАРОЛЕМ
================================================================================

9.1 На HQ-RTR (EcoRouter)
-------------------------
configure terminal
router ospf
 network 10.0.0.0/30 area 0
 network 192.168.10.0/27 area 0
 network 192.168.10.32/28 area 0
 network 192.168.10.48/29 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret
 exit
 exit
write

9.2 На HQ-RTR (Linux с FRR)
---------------------------
# Редактируем /etc/frr/frr.conf
nano /etc/frr/frr.conf

# Добавляем:
router ospf
 ospf router-id 192.168.10.1
 network 10.0.0.0/30 area 0
 network 192.168.10.0/27 area 0
 network 192.168.10.32/28 area 0
 network 192.168.10.48/29 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret

# Перезапускаем FRR
systemctl restart frr

9.3 На BR-RTR (аналогично, со своими сетями)
--------------------------------------------
router ospf
 network 10.0.0.0/30 area 0
 network 192.168.20.0/28 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret

9.4 Проверка OSPF
-----------------
# На EcoRouter:
show ip ospf neighbor
show ip route ospf

# На Linux:
vtysh -c "show ip ospf neighbor"
vtysh -c "show ip route ospf"

# Должны увидеть маршруты из другого офиса

================================================================================
10. ШАГ 8: NAT НА HQ-RTR И BR-RTR
================================================================================

10.1 На HQ-RTR
--------------
# Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT для офиса HQ (все пакеты из 192.168.10.0/24 через интерфейс к ISP)
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE

# Разрешаем форвардинг
iptables -A FORWARD -i eth0 -o eth0.100 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth0.200 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth0.999 -j ACCEPT
iptables -A FORWARD -i eth0.100 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0.200 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0.999 -o eth0 -j ACCEPT

10.2 На BR-RTR
--------------
# Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT для офиса BR
iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -o eth0 -j MASQUERADE

10.3 Проверка
-------------
# С HQ-CLI (или любой машины в HQ) пингуем 8.8.8.8
ping 8.8.8.8

================================================================================
11. ШАГ 9: DHCP ДЛЯ HQ-CLI (НА HQ-RTR)
================================================================================

11.1 Установка DHCP-сервера (на Альт Linux)
-------------------------------------------
apt-get install dhcp-server

11.2 Конфигурация /etc/dhcp/dhcpd.conf
--------------------------------------
nano /etc/dhcp/dhcpd.conf

# Содержимое:
subnet 192.168.10.32 netmask 255.255.255.240 {
    range 192.168.10.34 192.168.10.46;
    option routers 192.168.10.33;
    option domain-name-servers 192.168.10.2;
    option domain-name "au-team.irpo";
}

# Примечание: адрес маршрутизатора (192.168.10.33) исключен из диапазона автоматически

11.3 Запуск DHCP-сервера
------------------------
systemctl start dhcpd
systemctl enable dhcpd

11.4 Настройка HQ-CLI как DHCP-клиента
--------------------------------------
# На HQ-CLI:
dhclient eth0

# Проверяем полученный адрес
ip addr show eth0

================================================================================
12. ШАГ 10: DNS-СЕРВЕР НА HQ-SRV
================================================================================

12.1 Установка BIND
-------------------
apt-get install bind

12.2 Конфигурация /etc/named.conf
---------------------------------
nano /etc/named.conf

# Содержимое:
options {
    directory "/var/named";
    forwarders { 77.88.8.7; 77.88.8.3; };
    recursion yes;
    allow-query { any; };
};

zone "au-team.irpo" IN {
    type master;
    file "au-team.irpo.zone";
};

zone "10.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.10.rev";
};

zone "20.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.20.rev";
};

12.3 Прямая зона /var/named/au-team.irpo.zone
---------------------------------------------
nano /var/named/au-team.irpo.zone

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

hq-rtr  IN  A   192.168.10.1
br-rtr  IN  A   192.168.20.1
hq-srv  IN  A   192.168.10.2
hq-cli  IN  A   192.168.10.34
br-srv  IN  A   192.168.20.2
docker  IN  A   172.16.1.2
web     IN  A   172.16.2.2

12.4 Обратная зона для HQ /var/named/192.168.10.rev
---------------------------------------------------
nano /var/named/192.168.10.rev

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

1   IN  PTR hq-rtr.au-team.irpo.
2   IN  PTR hq-srv.au-team.irpo.
34  IN  PTR hq-cli.au-team.irpo.

12.5 Обратная зона для BR /var/named/192.168.20.rev
---------------------------------------------------
nano /var/named/192.168.20.rev

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

1   IN  PTR br-rtr.au-team.irpo.
2   IN  PTR br-srv.au-team.irpo.

12.6 Запуск DNS-сервера
-----------------------
systemctl start named
systemctl enable named

12.7 Проверка DNS
-----------------
# Проверяем прямые записи
nslookup hq-rtr.au-team.irpo 127.0.0.1
nslookup br-srv.au-team.irpo 127.0.0.1

# Проверяем обратные записи
nslookup 192.168.10.1 127.0.0.1
nslookup 192.168.20.2 127.0.0.1

================================================================================
13. ШАГ 11: НАСТРОЙКА ЧАСОВОГО ПОЯСА
================================================================================

На ВСЕХ устройствах (HQ-RTR, BR-RTR, HQ-SRV, BR-SRV, HQ-CLI, ISP):

timedatectl set-timezone Europe/Moscow
timedatectl status

================================================================================
14. ШАГ 12: ОФОРМЛЕНИЕ ОТЧЕТА
================================================================================

Создаем файл: ФамилияУчастникаМодуль1.docx (или .pdf)

СТРУКТУРА ОТЧЕТА:

1. ТИТУЛЬНАЯ СТРАНИЦА
   - Название учебного заведения
   - Демонстрационный экзамен по специальности 09.02.06
   - Модуль 1: Настройка сетевой инфраструктуры
   - ФИО участника, группа
   - Дата проведения

2. ТАБЛИЦА IP-АДРЕСОВ (как в Приложении 3)
   Копируем таблицу из шага 3.3

3. НАСТРОЙКА VLAN
   - Скриншот вывода команд: ip link show | grep vlan
   - Скриншот: ip addr show | grep "eth0\."
   - Текстовое описание: какие VLAN созданы и какие сети им назначены

4. НАСТРОЙКА ТУННЕЛЯ
   - Скриншот: ip tunnel show
   - Скриншот: ping 10.0.0.2 (с HQ-RTR)
   - Команды, которые использовались

5. ДИНАМИЧЕСКАЯ МАРШРУТИЗАЦИЯ (OSPF)
   - Скриншот: show ip ospf neighbor (или vtysh -c "show ip ospf neighbor")
   - Скриншот: show ip route ospf
   - Конфигурация OSPF с паролем (текст или скриншот)

6. NAT
   - Скриншот: iptables -t nat -L -n -v
   - Скриншот: ping 8.8.8.8 с HQ-CLI

7. DHCP
   - Конфиг /etc/dhcp/dhcpd.conf (текст или скриншот)
   - Скриншот: ip addr show eth0 на HQ-CLI (показать полученный адрес)

8. DNS
   - Скриншот: nslookup hq-rtr.au-team.irpo 127.0.0.1
   - Скриншот: nslookup 192.168.10.1 127.0.0.1
   - Скриншот: nslookup docker.au-team.irpo 127.0.0.1
   - Скриншот: nslookup web.au-team.irpo 127.0.0.1

9. ЧАСОВОЙ ПОЯС
   - Скриншот: timedatectl status (на любом устройстве)

================================================================================
15. ФИНАЛЬНЫЙ ЧЕК-ЛИСТ
================================================================================

Перед сдачей экзамена проверьте:

[ ] Все устройства имеют правильные имена (FQDN)
[ ] VLAN'ы работают: HQ-SRV и HQ-CLI в разных подсетях
[ ] HQ-SRV и HQ-CLI пингуют свои шлюзы
[ ] SSH работает на порту 2026, доступ только для sshuser
[ ] Баннер "Authorized access only" появляется при подключении SSH
[ ] Туннель поднят: ip tunnel show, пинг 10.0.0.2 работает
[ ] OSPF соседи установлены, маршруты из другого офиса видны
[ ] NAT работает: пинг 8.8.8.8 с HQ-CLI и BR-SRV успешен
[ ] DHCP выдал адрес HQ-CLI из правильного диапазона
[ ] DNS резолвит все A-записи из таблицы 3
[ ] DNS резолвит обратные PTR-записи
[ ] Часовой пояс на всех устройствах: Europe/Moscow
[ ] Отчет сохранен на рабочем столе с правильным именем
[ ] В отчете все скриншоты читаемые (кадрированы)

================================================================================
КОНЕЦ ШПАРГАЛКИ
================================================================================================================================================================
        МОДУЛЬ 1: ПОЛНАЯ ПОШАГОВАЯ ИНСТРУКЦИЯ (КОД 09.02.06-1-2026)
        Файл: ФамилияУчастникаМодуль1.txt
        Версия: 1.0 (подробная шпаргалка)
================================================================================

СОДЕРЖАНИЕ:
1.  Вводная информация
2.  Тайм-менеджмент
3.  Шаг 1: Базовая настройка устройств (имена, IP)
4.  Шаг 2: Настройка VLAN на HQ-RTR
5.  Шаг 3: Создание локальных учетных записей
6.  Шаг 4: Настройка ISP (доступ в интернет)
7.  Шаг 5: Настройка безопасного SSH на серверах
8.  Шаг 6: Настройка IP-туннеля (GRE)
9.  Шаг 7: Динамическая маршрутизация (OSPF) с паролем
10. Шаг 8: NAT на HQ-RTR и BR-RTR
11. Шаг 9: DHCP для HQ-CLI
12. Шаг 10: DNS-сервер на HQ-SRV
13. Шаг 11: Настройка часового пояса
14. Шаг 12: Оформление отчета
15. Финальный чек-лист

================================================================================
1. ВВОДНАЯ ИНФОРМАЦИЯ
================================================================================
Время выполнения: 1 час (60 минут)
Что нужно сдать: рабочая сеть + отчет
Имя файла отчета: ФамилияУчастникаМодуль1.docx или .pdf

Топология:
- Офис HQ: HQ-RTR, HQ-SRV, HQ-CLI (VLAN 100, 200, 999)
- Офис BR: BR-RTR, BR-SRV
- ISP: соединяет HQ-RTR и BR-RTR с интернетом

Дистрибутивы:
- HQ-SRV, BR-SRV, HQ-CLI: Альт Сервер / Альт Рабочая станция
- HQ-RTR, BR-RTR: EcoRouter (или Альт JeOS с FRR)
- ISP: Альт JeOS

================================================================================
2. ТАЙМ-МЕНЕДЖМЕНТ (60 МИНУТ)
================================================================================
| Этап | Действие                                     | Время  |
|------|----------------------------------------------|--------|
| 1    | Базовая настройка (имена, IP-адреса)         | 10 мин |
| 2    | VLAN на HQ-RTR                               | 5 мин  |
| 3    | Пользователи (sshuser, net_admin)            | 5 мин  |
| 4    | ISP (DHCP, NAT)                              | 5 мин  |
| 5    | SSH на серверах (порт 2026, баннер)          | 5 мин  |
| 6    | IP-туннель (GRE)                             | 5 мин  |
| 7    | OSPF с паролем                               | 5 мин  |
| 8    | NAT на HQ-RTR и BR-RTR                       | 5 мин  |
| 9    | DHCP для HQ-CLI                              | 5 мин  |
| 10   | DNS-сервер на HQ-SRV                         | 10 мин |
| 11   | Часовой пояс                                 | 2 мин  |
| 12   | Отчет (скриншоты, таблицы)                   | 8 мин  |

================================================================================
3. ШАГ 1: БАЗОВАЯ НАСТРОЙКА УСТРОЙСТВ
================================================================================

3.1 Настройка имени хоста (FQDN)
--------------------------------
На КАЖДОМ устройстве выполняем команду:

hostnamectl set-hostname <имя>.au-team.irpo

Примеры:
- HQ-RTR:   hostnamectl set-hostname hq-rtr.au-team.irpo
- BR-RTR:   hostnamectl set-hostname br-rtr.au-team.irpo
- HQ-SRV:   hostnamectl set-hostname hq-srv.au-team.irpo
- BR-SRV:   hostnamectl set-hostname br-srv.au-team.irpo
- HQ-CLI:   hostnamectl set-hostname hq-cli.au-team.irpo
- ISP:      hostnamectl set-hostname isp

Проверка: hostnamectl

3.2 Расчет IP-адресации (VLSM)
------------------------------
Требования задания:
- HQ-SRV (VLAN 100): ≤ 32 адреса -> маска /27 (32 адреса)
- HQ-CLI (VLAN 200): ≥ 16 адресов -> маска /28 (16 адресов)
- Управление (VLAN 999): ≤ 8 адресов -> маска /29 (8 адресов)
- BR-SRV: ≤ 16 адресов -> маска /28

ВЫБРАННЫЕ СЕТИ (из RFC 1918):

| VLAN/сеть      | Сеть           | Маска | Диапазон               | Шлюз            |
|----------------|----------------|-------|------------------------|-----------------|
| VLAN 100       | 192.168.10.0   | /27   | 192.168.10.1-30        | 192.168.10.1    |
| VLAN 200       | 192.168.10.32  | /28   | 192.168.10.33-46       | 192.168.10.33   |
| VLAN 999       | 192.168.10.48  | /29   | 192.168.10.49-54       | 192.168.10.49   |
| BR-SRV         | 192.168.20.0   | /28   | 192.168.20.1-14        | 192.168.20.1    |

3.3 Таблица адресов (заполнить в отчете)
----------------------------------------
| Имя устройства | IP-адрес                                    | Шлюз по умолчанию |
|----------------|---------------------------------------------|-------------------|
| HQ-RTR         | 192.168.10.1/27, 192.168.10.33/28, 192.168.10.49/29 | —                 |
| BR-RTR         | 192.168.20.1/28                             | —                 |
| HQ-SRV         | 192.168.10.2/27                             | 192.168.10.1      |
| BR-SRV         | 192.168.20.2/28                             | 192.168.20.1      |
| HQ-CLI         | 192.168.10.34/28                            | 192.168.10.33     |

3.4 Назначение IP-адресов (временные, до настройки VLAN)
--------------------------------------------------------
На HQ-RTR (на физический интерфейс eth0, позже заменим на VLAN):
  ip addr add 192.168.10.1/27 dev eth0
  ip addr add 192.168.10.33/28 dev eth0
  ip addr add 192.168.10.49/29 dev eth0

На BR-RTR:
  ip addr add 192.168.20.1/28 dev eth0

На HQ-SRV:
  ip addr add 192.168.10.2/27 dev eth0
  ip route add default via 192.168.10.1

На BR-SRV:
  ip addr add 192.168.20.2/28 dev eth0
  ip route add default via 192.168.20.1

На HQ-CLI:
  ip addr add 192.168.10.34/28 dev eth0
  ip route add default via 192.168.10.33

================================================================================
4. ШАГ 2: НАСТРОЙКА VLAN НА HQ-RTR (С ИСПОЛЬЗОВАНИЕМ ОДНОГО ФИЗИЧЕСКОГО ПОРТА)
================================================================================

На HQ-RTR (предполагаем, что физический интерфейс называется eth0):

# Создаем VLAN-интерфейсы
ip link add link eth0 name eth0.100 type vlan id 100
ip link add link eth0 name eth0.200 type vlan id 200
ip link add link eth0 name eth0.999 type vlan id 999

# Включаем их
ip link set eth0.100 up
ip link set eth0.200 up
ip link set eth0.999 up

# Назначаем IP-адреса на VLAN-интерфейсы
ip addr add 192.168.10.1/27 dev eth0.100
ip addr add 192.168.10.33/28 dev eth0.200
ip addr add 192.168.10.49/29 dev eth0.999

# Удаляем старые IP с физического интерфейса (если были)
ip addr del 192.168.10.1/27 dev eth0
ip addr del 192.168.10.33/28 dev eth0
ip addr del 192.168.10.49/29 dev eth0

# Включаем IP-форвардинг (для маршрутизации между VLAN)
echo 1 > /proc/sys/net/ipv4/ip_forward

================================================================================
5. ШАГ 3: СОЗДАНИЕ ЛОКАЛЬНЫХ УЧЕТНЫХ ЗАПИСЕЙ
================================================================================

5.1 На HQ-SRV и BR-SRV (серверы)
--------------------------------
# Создаем пользователя sshuser с UID 2026
useradd -m -u 2026 sshuser
echo "sshuser:P@ssw0rd" | chpasswd

# Даем право sudo без пароля
echo "sshuser ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/sshuser
chmod 440 /etc/sudoers.d/sshuser

5.2 На HQ-RTR и BR-RTR (маршрутизаторы)
---------------------------------------
# Создаем пользователя net_admin
useradd -m net_admin
echo "net_admin:P@ssw0rd" | chpasswd

# Для EcoRouter - добавляем в группу администраторов
usermod -aG wheel net_admin

# Для Linux (Альт) - даем sudo без пароля
echo "net_admin ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/net_admin
chmod 440 /etc/sudoers.d/net_admin

================================================================================
6. ШАГ 4: НАСТРОЙКА ISP (ДОСТУП В ИНТЕРНЕТ)
================================================================================

ISP имеет:
- eth0 -> в магистрального провайдера (получает адрес по DHCP)
- eth1 -> к HQ-RTR (сеть 172.16.1.0/28)
- eth2 -> к BR-RTR (сеть 172.16.2.0/28)

Выполняем на ISP:

# 1. Интерфейс к провайдеру - получаем адрес по DHCP
dhclient eth0

# 2. Интерфейсы к офисам
ip addr add 172.16.1.1/28 dev eth1
ip addr add 172.16.2.1/28 dev eth2
ip link set eth1 up
ip link set eth2 up

# 3. Маршрут по умолчанию (из DHCP)
#    Получаем gateway из вывода: ip route show
#    Добавляем:
ip route add default via <gateway_полученный_от_dhcp>

# 4. Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# 5. Настраиваем NAT (динамическая трансляция портов)
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 6. Разрешаем форвардинг между интерфейсами
iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth2 -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth2 -o eth0 -j ACCEPT

# 7. Сохраняем правила (для Альт Linux)
iptables-save > /etc/sysconfig/iptables

================================================================================
7. ШАГ 5: НАСТРОЙКА БЕЗОПАСНОГО SSH НА HQ-SRV И BR-SRV
================================================================================

Выполняем НА КАЖДОМ СЕРВЕРЕ (HQ-SRV и BR-SRV):

# 1. Редактируем конфиг SSH
nano /etc/ssh/sshd_config

# Вносим изменения:
Port 2026
PermitRootLogin no
AllowUsers sshuser
MaxAuthTries 2
Banner /etc/ssh/banner.txt

# 2. Создаем баннер
echo "Authorized access only" > /etc/ssh/banner.txt

# 3. Перезапускаем SSH
systemctl restart sshd

# 4. Проверяем, что слушается порт 2026
ss -tlnp | grep 2026

================================================================================
8. ШАГ 6: НАСТРОЙКА IP-ТУННЕЛЯ (GRE)
================================================================================

8.1 На HQ-RTR
-------------
# IP-адреса для туннеля
# local = адрес ISP-интерфейса HQ-RTR (172.16.1.2)
# remote = адрес ISP-интерфейса BR-RTR (172.16.2.2)

ip tunnel add tun0 mode gre remote 172.16.2.2 local 172.16.1.2
ip addr add 10.0.0.1/30 dev tun0
ip link set tun0 up

8.2 На BR-RTR
-------------
ip tunnel add tun0 mode gre remote 172.16.1.2 local 172.16.2.2
ip addr add 10.0.0.2/30 dev tun0
ip link set tun0 up

8.3 Проверка
------------
# С HQ-RTR пингуем BR-RTR через туннель
ping 10.0.0.2

================================================================================
9. ШАГ 7: ДИНАМИЧЕСКАЯ МАРШРУТИЗАЦИЯ (OSPF) С ПАРОЛЕМ
================================================================================

9.1 На HQ-RTR (EcoRouter)
-------------------------
configure terminal
router ospf
 network 10.0.0.0/30 area 0
 network 192.168.10.0/27 area 0
 network 192.168.10.32/28 area 0
 network 192.168.10.48/29 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret
 exit
 exit
write

9.2 На HQ-RTR (Linux с FRR)
---------------------------
# Редактируем /etc/frr/frr.conf
nano /etc/frr/frr.conf

# Добавляем:
router ospf
 ospf router-id 192.168.10.1
 network 10.0.0.0/30 area 0
 network 192.168.10.0/27 area 0
 network 192.168.10.32/28 area 0
 network 192.168.10.48/29 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret

# Перезапускаем FRR
systemctl restart frr

9.3 На BR-RTR (аналогично, со своими сетями)
--------------------------------------------
router ospf
 network 10.0.0.0/30 area 0
 network 192.168.20.0/28 area 0
 area 0 authentication message-digest
 interface tun0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 OSPFsecret

9.4 Проверка OSPF
-----------------
# На EcoRouter:
show ip ospf neighbor
show ip route ospf

# На Linux:
vtysh -c "show ip ospf neighbor"
vtysh -c "show ip route ospf"

# Должны увидеть маршруты из другого офиса

================================================================================
10. ШАГ 8: NAT НА HQ-RTR И BR-RTR
================================================================================

10.1 На HQ-RTR
--------------
# Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT для офиса HQ (все пакеты из 192.168.10.0/24 через интерфейс к ISP)
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -o eth0 -j MASQUERADE

# Разрешаем форвардинг
iptables -A FORWARD -i eth0 -o eth0.100 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth0.200 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth0.999 -j ACCEPT
iptables -A FORWARD -i eth0.100 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0.200 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0.999 -o eth0 -j ACCEPT

10.2 На BR-RTR
--------------
# Включаем IP-форвардинг
echo 1 > /proc/sys/net/ipv4/ip_forward

# NAT для офиса BR
iptables -t nat -A POSTROUTING -s 192.168.20.0/24 -o eth0 -j MASQUERADE

10.3 Проверка
-------------
# С HQ-CLI (или любой машины в HQ) пингуем 8.8.8.8
ping 8.8.8.8

================================================================================
11. ШАГ 9: DHCP ДЛЯ HQ-CLI (НА HQ-RTR)
================================================================================

11.1 Установка DHCP-сервера (на Альт Linux)
-------------------------------------------
apt-get install dhcp-server

11.2 Конфигурация /etc/dhcp/dhcpd.conf
--------------------------------------
nano /etc/dhcp/dhcpd.conf

# Содержимое:
subnet 192.168.10.32 netmask 255.255.255.240 {
    range 192.168.10.34 192.168.10.46;
    option routers 192.168.10.33;
    option domain-name-servers 192.168.10.2;
    option domain-name "au-team.irpo";
}

# Примечание: адрес маршрутизатора (192.168.10.33) исключен из диапазона автоматически

11.3 Запуск DHCP-сервера
------------------------
systemctl start dhcpd
systemctl enable dhcpd

11.4 Настройка HQ-CLI как DHCP-клиента
--------------------------------------
# На HQ-CLI:
dhclient eth0

# Проверяем полученный адрес
ip addr show eth0

================================================================================
12. ШАГ 10: DNS-СЕРВЕР НА HQ-SRV
================================================================================

12.1 Установка BIND
-------------------
apt-get install bind

12.2 Конфигурация /etc/named.conf
---------------------------------
nano /etc/named.conf

# Содержимое:
options {
    directory "/var/named";
    forwarders { 77.88.8.7; 77.88.8.3; };
    recursion yes;
    allow-query { any; };
};

zone "au-team.irpo" IN {
    type master;
    file "au-team.irpo.zone";
};

zone "10.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.10.rev";
};

zone "20.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.20.rev";
};

12.3 Прямая зона /var/named/au-team.irpo.zone
---------------------------------------------
nano /var/named/au-team.irpo.zone

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

hq-rtr  IN  A   192.168.10.1
br-rtr  IN  A   192.168.20.1
hq-srv  IN  A   192.168.10.2
hq-cli  IN  A   192.168.10.34
br-srv  IN  A   192.168.20.2
docker  IN  A   172.16.1.2
web     IN  A   172.16.2.2

12.4 Обратная зона для HQ /var/named/192.168.10.rev
---------------------------------------------------
nano /var/named/192.168.10.rev

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

1   IN  PTR hq-rtr.au-team.irpo.
2   IN  PTR hq-srv.au-team.irpo.
34  IN  PTR hq-cli.au-team.irpo.

12.5 Обратная зона для BR /var/named/192.168.20.rev
---------------------------------------------------
nano /var/named/192.168.20.rev

# Содержимое:
$TTL 86400
@   IN  SOA hq-srv.au-team.irpo. admin.au-team.irpo. (
    2026032101 ; serial
    3600       ; refresh
    1800       ; retry
    604800     ; expire
    86400      ; minimum
)
@   IN  NS  hq-srv.au-team.irpo.

1   IN  PTR br-rtr.au-team.irpo.
2   IN  PTR br-srv.au-team.irpo.

12.6 Запуск DNS-сервера
-----------------------
systemctl start named
systemctl enable named

12.7 Проверка DNS
-----------------
# Проверяем прямые записи
nslookup hq-rtr.au-team.irpo 127.0.0.1
nslookup br-srv.au-team.irpo 127.0.0.1

# Проверяем обратные записи
nslookup 192.168.10.1 127.0.0.1
nslookup 192.168.20.2 127.0.0.1

================================================================================
13. ШАГ 11: НАСТРОЙКА ЧАСОВОГО ПОЯСА
================================================================================

На ВСЕХ устройствах (HQ-RTR, BR-RTR, HQ-SRV, BR-SRV, HQ-CLI, ISP):

timedatectl set-timezone Europe/Moscow
timedatectl status

================================================================================
14. ШАГ 12: ОФОРМЛЕНИЕ ОТЧЕТА
================================================================================

Создаем файл: ФамилияУчастникаМодуль1.docx (или .pdf)

СТРУКТУРА ОТЧЕТА:

1. ТИТУЛЬНАЯ СТРАНИЦА
   - Название учебного заведения
   - Демонстрационный экзамен по специальности 09.02.06
   - Модуль 1: Настройка сетевой инфраструктуры
   - ФИО участника, группа
   - Дата проведения

2. ТАБЛИЦА IP-АДРЕСОВ (как в Приложении 3)
   Копируем таблицу из шага 3.3

3. НАСТРОЙКА VLAN
   - Скриншот вывода команд: ip link show | grep vlan
   - Скриншот: ip addr show | grep "eth0\."
   - Текстовое описание: какие VLAN созданы и какие сети им назначены

4. НАСТРОЙКА ТУННЕЛЯ
   - Скриншот: ip tunnel show
   - Скриншот: ping 10.0.0.2 (с HQ-RTR)
   - Команды, которые использовались

5. ДИНАМИЧЕСКАЯ МАРШРУТИЗАЦИЯ (OSPF)
   - Скриншот: show ip ospf neighbor (или vtysh -c "show ip ospf neighbor")
   - Скриншот: show ip route ospf
   - Конфигурация OSPF с паролем (текст или скриншот)

6. NAT
   - Скриншот: iptables -t nat -L -n -v
   - Скриншот: ping 8.8.8.8 с HQ-CLI

7. DHCP
   - Конфиг /etc/dhcp/dhcpd.conf (текст или скриншот)
   - Скриншот: ip addr show eth0 на HQ-CLI (показать полученный адрес)

8. DNS
   - Скриншот: nslookup hq-rtr.au-team.irpo 127.0.0.1
   - Скриншот: nslookup 192.168.10.1 127.0.0.1
   - Скриншот: nslookup docker.au-team.irpo 127.0.0.1
   - Скриншот: nslookup web.au-team.irpo 127.0.0.1

9. ЧАСОВОЙ ПОЯС
   - Скриншот: timedatectl status (на любом устройстве)

================================================================================
15. ФИНАЛЬНЫЙ ЧЕК-ЛИСТ
================================================================================

Перед сдачей экзамена проверьте:

[ ] Все устройства имеют правильные имена (FQDN)
[ ] VLAN'ы работают: HQ-SRV и HQ-CLI в разных подсетях
[ ] HQ-SRV и HQ-CLI пингуют свои шлюзы
[ ] SSH работает на порту 2026, доступ только для sshuser
[ ] Баннер "Authorized access only" появляется при подключении SSH
[ ] Туннель поднят: ip tunnel show, пинг 10.0.0.2 работает
[ ] OSPF соседи установлены, маршруты из другого офиса видны
[ ] NAT работает: пинг 8.8.8.8 с HQ-CLI и BR-SRV успешен
[ ] DHCP выдал адрес HQ-CLI из правильного диапазона
[ ] DNS резолвит все A-записи из таблицы 3
[ ] DNS резолвит обратные PTR-записи
[ ] Часовой пояс на всех устройствах: Europe/Moscow
[ ] Отчет сохранен на рабочем столе с правильным именем
[ ] В отчете все скриншоты читаемые (кадрированы)

================================================================================
КОНЕЦ ШПАРГАЛКИ
================================================================================
