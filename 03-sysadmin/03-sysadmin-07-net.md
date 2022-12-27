## 03-sysadmin-07-net
1. Проверьте список доступных сетевых интерфейсов на вашем компьютере. Какие команды есть для этого в Linux и в Windows?
```bash
# Это мой домашний сервер на Debian, так что интерфейсов там полно.
❯ ip -br -c link
lo               UNKNOWN        00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eno1             UP             b4:7a:f1:ae:5e:32 <BROADCAST,MULTICAST,UP,LOWER_UP>
eno2             DOWN           b4:7a:f1:ae:5e:33 <NO-CARRIER,BROADCAST,MULTICAST,UP>
eno3             DOWN           b4:7a:f1:ae:5e:34 <BROADCAST,MULTICAST>
eno4             DOWN           b4:7a:f1:ae:5e:35 <BROADCAST,MULTICAST>
ovs-system       DOWN           62:0d:25:9a:9e:0d <BROADCAST,MULTICAST>
vmbr0            UNKNOWN        b4:7a:f1:ae:5e:32 <BROADCAST,MULTICAST,UP,LOWER_UP>
k8s_vlan         UNKNOWN        12:f8:06:51:b3:9a <BROADCAST,MULTICAST,UP,LOWER_UP>
bond0            UNKNOWN        96:1c:59:2f:41:db <BROADCAST,MULTICAST,UP,LOWER_UP>
br-0d83797b9c39  UP             02:42:d1:2d:a0:0f <BROADCAST,MULTICAST,UP,LOWER_UP>
br-4d94c796c091  UP             02:42:b7:d3:d8:ba <BROADCAST,MULTICAST,UP,LOWER_UP>
docker0          UP             02:42:69:33:a1:4d <BROADCAST,MULTICAST,UP,LOWER_UP>
br-22ad66093295  UP             02:42:ae:d9:bf:2e <BROADCAST,MULTICAST,UP,LOWER_UP>
br-71071f0096c0  UP             02:42:c7:62:c9:d8 <BROADCAST,MULTICAST,UP,LOWER_UP>
br-7e535f68a39a  UP             02:42:36:e9:2a:5f <BROADCAST,MULTICAST,UP,LOWER_UP>
br-e02bd5997eb0  UP             02:42:fa:95:9f:c5 <BROADCAST,MULTICAST,UP,LOWER_UP>
br-f83ec4330c6d  UP             02:42:46:93:74:a6 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth745884f@if20 UP             a6:90:d9:30:eb:92 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethe760b00@if24 UP             8a:49:f4:f1:94:c1 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth4cface5@if1561 UP             5a:cd:fd:22:0e:a9 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethfd1e562@if26 UP             06:12:1d:e7:d2:2f <BROADCAST,MULTICAST,UP,LOWER_UP>
veth3430ea2@if30 UP             fa:ad:a2:98:24:e2 <BROADCAST,MULTICAST,UP,LOWER_UP>
vetha1bdb07@if36 UP             86:5d:f3:e4:2e:ac <BROADCAST,MULTICAST,UP,LOWER_UP>
veth53a3620@if38 UP             8a:2e:c2:f4:f4:50 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth2ddb007@if40 UP             7a:bc:af:fa:de:60 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethe9e2958@if42 UP             0e:9f:4f:1a:59:1e <BROADCAST,MULTICAST,UP,LOWER_UP>
vethe63a51a@if44 UP             f6:61:76:ec:03:4a <BROADCAST,MULTICAST,UP,LOWER_UP>
vethe1ecd34@if46 UP             52:8f:29:52:64:d3 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth9d2fd03@if48 UP             02:68:cb:38:c1:4b <BROADCAST,MULTICAST,UP,LOWER_UP>
veth19d1ee2@if52 UP             d6:c5:7a:0b:df:36 <BROADCAST,MULTICAST,UP,LOWER_UP>
tap101i0         UNKNOWN        46:19:4d:c0:64:5f <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP>
fwbr101i0        UP             6e:37:af:f0:96:45 <BROADCAST,MULTICAST,UP,LOWER_UP>
fwln101o0        UNKNOWN        b6:c5:3d:b9:99:7f <BROADCAST,MULTICAST,UP,LOWER_UP>
veth3d8f4b4@if61 UP             3e:e1:50:7c:4e:41 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth0988b8e@if63 UP             9a:77:42:6e:4a:cc <BROADCAST,MULTICAST,UP,LOWER_UP>
br-29a00f63b4a7  UP             02:42:85:99:1e:10 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethbf0f74a@if87 UP             e6:2e:d8:bf:b4:80 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth3c18a3a@if1113 UP             36:3e:f2:3a:7b:88 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth42b8755@if1115 UP             9a:5c:e4:15:44:06 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth0eafa54@if1117 UP             3e:37:da:72:87:e2 <BROADCAST,MULTICAST,UP,LOWER_UP>
vethc3b2739@if132 UP             f2:93:c8:64:c3:cf <BROADCAST,MULTICAST,UP,LOWER_UP>
vetha752907@if137 UP             6a:f5:57:0f:c3:24 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth3443a31@if146 UP             52:cd:9b:cd:98:d1 <BROADCAST,MULTICAST,UP,LOWER_UP>
veth6376a3a@if1217 UP             5a:9a:3d:8e:06:3f <BROADCAST,MULTICAST,UP,LOWER_UP>
```
```powershell
# В случае с Windows поможет
ipconfig /all
```
2. Какой протокол используется для распознавания соседа по сетевому интерфейсу? Какой пакет и команды есть в Linux для этого?
```bash
# Протокол LLDP
❯ apt-cache search ^lldpd$
lldpd - implementation of IEEE 802.1ab (LLDP)
```
3. Какая технология используется для разделения L2 коммутатора на несколько виртуальных сетей? Какой пакет и команды есть в Linux для этого? Приведите пример конфига.
```bash
# VLAN, у меня как раз есть такой между виртуалками с кубером, правда на Open vSwitch
auto k8s_vlan
iface k8s_vlan inet static
        address 192.158.100.1/24
        ovs_type OVSIntPort
        ovs_bridge vmbr0
        ovs_options tag=100
# Впрочем, можно сделать это сделать и в каком-нибудь netplan
vlans: 
    vlan-test:
      id: 101
      link: vmbr0
      dhcp4: no
      addresses: [192.158.200.2/24]
      gateway: 192.158.200.1
      routes:
        - to: 192.158.200.2/24
          via: 192.158.200.1
          on-link: true
```
4. Какие типы агрегации интерфейсов есть в Linux? Какие опции есть для балансировки нагрузки? Приведите пример конфига.
```bash
# balance-rr - Последовательно кидает пакеты, с первого по последний интерфейс.
# active-backup - Один из интерфейсов активен. Если активный интерфейс выходит из строя (link down и т.д.), другой интерфейс заменяет активный. Не требует дополнительной настройки коммутатора
# balance-xor - Передачи распределяются между интерфейсами на основе формулы ((MAC-адрес источника) XOR (MAC-адрес получателя)) % число интерфейсов. Один и тот же интерфейс работает с определённым получателем. Режим даёт балансировку нагрузки и отказоустойчивость.
# broadcast - Все пакеты на все интерфейсы
# LACP (802.3ad) - Link Agregation — IEEE 802.3ad, требует от коммутатора настройки.
# balance-tlb - Входящие пакеты принимаются только активным сетевым интерфейсом, исходящий распределяется в зависимости от текущей загрузки каждого интерфейса. Не требует настройки коммутатора.
# balance-alb - Тоже самое что 5, только входящий трафик тоже распределяется между интерфейсами. Не требует настройки коммутатора, но интерфейсы должны уметь изменять MAC.
# В моём случае это LACP:
auto bond0
iface bond0 inet manual
        ovs_bonds eno1 eno2
        ovs_type OVSBond
        ovs_bridge vmbr0
        ovs_options lacp=active bond_mode=balance-tcp
```
5. Сколько IP адресов в сети с маской /29 ? Сколько /29 подсетей можно получить из сети с маской /24. Приведите несколько примеров /29 подсетей внутри сети 10.10.10.0/24.
```bash
# 8 адресов, для хостов доступно 6
# 32 сети можно создать внутри /24 префикса
# 10.10.10.72/29
# 10.10.10.160/29
# 10.10.10.216/29
```
6. Задача: вас попросили организовать стык между 2-мя организациями. Диапазоны 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 уже заняты. Из какой подсети допустимо взять частные IP адреса? Маску выберите из расчета максимум 40-50 хостов внутри подсети.
```bash
# Остается только 100.64.0.0/10, в которой можно взять любой понравившийся /26 префикс. Если все диапазоны заняты да еще и пересекаются - придется натиться.

```
7. Как проверить ARP таблицу в Linux, Windows? Как очистить ARP кеш полностью? Как из ARP таблицы удалить только один нужный IP?
```bash
arp -a # Посмотреть таблицу
ip -s -s neigh flush all # Дропнуть весь ARP кэш
arp -d address # Удалить один адрес
```
8. [*] Установите эмулятор EVE-ng.
Инструкция по установке - https://github.com/svmyasnikov/eve-ng
Выполните задания на lldp, vlan, bonding в эмуляторе EVE-ng. 
---