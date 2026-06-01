всем удачи
https://files.catbox.moe/ejw5iq.pdf запас
https://github.com/MemoryOfGood/DemoExamSSA2026 хз, мб не то
https://drive.google.com/drive/folders/1zLRcwNIyeG1Lq0EdzxuCIEHbzA1Md3yd?usp=drive_link запас если не скачивается с гитхаба
https://github.com/peplixmain/sasamba/ 2 модуль


<details>
  <summary>Как расчитать подсети</summary>
1. Как рассчитать подсети?
Выбор маски подсети (префикса) зависит от того, сколько устройств (адресов) вам нужно в этой сети. В задании указаны лимиты:
VLAN 100 (Серверы HQ): нужно до 64 адресов.
Расчет: Ближайшее подходящее число — 64. Это префикс /26 (маска 255.255.255.192)
.
Примечание: В версии 2026 года лимит снижен до 32 адресов — это префикс /27
.
VLAN 200 (Клиенты HQ): не менее 16 адресов.
Расчет: Для 16 адресов нужен префикс /28, но часто для клиентов берут стандартный /24 (254 адреса), чтобы точно хватило
.
VLAN 999 (Управление): до 8 адресов.
Расчет: Это префикс /29 (маска 255.255.255.248)
.
Сеть в сторону BR-SRV: до 16 или 32 адресов.
Расчет: Это /28 (для 16) или /27 (для 32)

На серверах и клиентах (ОС «Альт»)
Здесь используется система etcnet. Вы не просто вводите команду, а редактируете файлы в папке /etc/net/ifaces/<ИМЯ_ИНТЕРФЕЙСА>/
.
Узнайте имя интерфейса: введите ip a. Обычно это ens19, ens20 и т.д.
.
Настройте параметры (файл options):
vim /etc/net/ifaces/ens19/options
Должно быть: TYPE=eth и BOOTPROTO=static
.
Укажите IP-адрес (файл ipv4address):
echo "192.168.100.2/27" > /etc/net/ifaces/ens19/ipv4address
.
Укажите шлюз (файл ipv4route):
echo "default via 192.168.100.1" > /etc/net/ifaces/ens19/ipv4route
.
Перезапустите сеть: systemctl restart network
.
Б. На маршрутизаторах (EcoRouterOS)
Здесь логика сложнее: сначала создаем виртуальный интерфейс («мозг»), а потом привязываем его к физическому порту («руки»)
.
Создаем интерфейс:
Привязываем к порту (например, порт ge1):
Маршрут по умолчанию (в сторону ISP): ip route 0.0.0.0/0 172.16.1.1

</details>
______________________________________________
<details>
  <summary>старое, возможно не правильно</summary>

1. в ISP
   >hostnamectl hostname isp; exec bash
2. в (br-srv) (hq-cli) (hq-srv).au-team.irpo; exec bash

3. настройка команд на RTR
     >en
     >sh ru ( проверить на преднастрой) 
     >hostname (br-rtr,hq-rtr)
    >ip domain-name au-team.irpo
    >exit 
    >wr mem
4. в ISP
    >ip -c --br a
    (ели не настроины ip см. ниже)
    >mc /etc/net/ifaces/
    mkdir(создать папку)
5. если не преднастройки папок, заходим в созданную папку и копируем настройки на ens4 не нужен дхсп, так как оно направлено в сервак.
BOOTPROTO=STATIC
SYSTEMD_BOOTPRO=STATIC
прописать айпишник :shift+f4
172.16.1.1/29
F10 , имя файла ipv4address

6. systemctl restart network-обновляет сеть. 
(проверка айпишников ip -c --br a)

7. HQ-SRV 
(анологично, вход в ifaces)
  
8. если нету адреса в ens, создаём файл и прописываем 
10.10.100.2/29

9. новый файл, default via 10.10.100.1(название ipv4route)

10. новый файл, задать дефолтный днс сервер.
(search au-team.irpo) - эта конструкция только для hq-srv
(nameserver 8.8.8.8) ,

11. для br-srv указать  айпи hq-srv
имя файла (resolv.conf)
systemctl restart network
ip -c --br a
ip -c --br r

12. HQ-RTR
sh ru
interface isp
ip address 172.16.1.2/29
port ge0
port ge0 service-instance (любое название)
encapsulation untagged
connect ip interface isp


13. в случае отсутствия vlan

айпи адреса по vlan 
vl100 10.10.100.1
vl200 10.10.200.1
vl999 10.10.30.1
interface vl100
ip address 10.10.100.1/29
port ge1
service-instance vl100
encapsulation dot1q 100
rewrite pop 1 
connect ip interface vl100
</details>
______________________________________________
<details>
  <summary>новое, более менее правильно</summary>
# Настройка сети

## ISP

```bash
root toor
hostnamectl set-hostname ISP
exec bash
```

## HQ Router

```bash
admin admin
en
conf t
hostname hq-rtr
ip domain-name au-team.irpo
exit
write memory
```

## BR Router

```bash
admin admin
en
conf t
hostname br-rtr
ip domain-name au-team.irpo
exit
write memory
```

## HQ Switch

```bash
root toor
ovs-vsctl del-br hq-sw
ovs-vsctl add-br hq-sw
ovs-vsctl add-port hq-sw ens3 trunk=111,211,811
ovs-vsctl add-port hq-sw ens4 tag=111
ovs-vsctl add-port hq-sw ens5 tag=211
# ВЛАНЫ МОГУТ БЫТЬ ДРУГИМИ
```

## HQ Server

```bash
root toor
ip a -c --br
cd /etc/net/ifaces/ensВОЗМОЖНО4
mc
# шаг1: shift+f4 -> ~10.10.100.2/27 (хз я не знаю точно такой ip или другой) нажимаем f2 имя=ipv4address 
# шаг2: mc shift+f4 -> default via 10.10.100.1 f2 имя=ipv4rooting
# шаг3: mc shift+f4 -> nameserver 77.88.8.8, search au-team.irpo f2 имя=resolve.conf
systemctl restart network
```

## Переназначение VLAN на HQ Router

```bash
admin admin
en
conf t
port ge1
no service-instance vl100
no service-instance vl200
no service-instance vl999
service-instance vl111
encapsulation dot1q 111
rewrite pop 1
connect ip interface vl100
service-instance vl211
encapsulation dot1q 211
rewrite pop 1
connect ip interface vl200
service-instance vl811
encapsulation dot1q 811
rewrite pop 1
connect ip interface vl999
```

## Настройка интерфейсов и туннелей
hq-rtr
```bash
en
conf t
interface ISP
ip address 172.16.10.2/28
exit
port ge0 # 
service-instance ISP
encapsulation untagged
connect ip interface ISP
exit

port <портксерверам> # например ge0
service-instance vlan111
encapsulation dot1q 111
connect bridge <bridge>
exit

interface tunnel.1
ip address 10.10.10.1/30
tunnel source <внешний_IP> котороый в сторону ips
tunnel destination <BR_IP>

router ospf 1
router-id <IP>
area 0 authentication message-digest
network tunnel.1 area 0
```
## HQ cli
```bash
mv /etc/net/ifaces/ens18 /etc/net/ifaces/ens3
systemctl restart network 
нужно переименовать ens18 в ens3
bootproto static поменять на DHCP4 в. options
systemctl restart network
ip -c a
проверяешь что ip по DHCP4
```


# NAT ISP
```
iptables -t nat -A POSTROUTING -s 172.16.1.0/28 -o <внешний_интерфейс> -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.2.0/28 -o <внешний_интерфейс> -j MASQUERADE
iptables-save >> /etc/sysconfig/iptables
systemctl enable --now iptables

```

#HQ-SRV,BR-SRV
```
usermod -u 2011 sshuser
меняем id юзера и 
passwd sshuser "enter"
пароль пишем
gpasswd -a sshuser wheel

```
#бинды
```
apt-get update && apt-get install bind bind-utils -y
mcedit /var/lib/bind/etc/options.conf
listen-on port 53 { any; };
forwarders { 77.88.8.8; };
allow-query { any; };

mcedit /var/lib/bind/etc/rfc1912.conf
zone "au-team.irpo" { type master; file "au-team.irpo"; };
zone "100.10.10.in-addr.arpa" { type master; file "10.10.100.inaddr.arpa"; };
zone "200.10.10.in-addr.arpa" { type master; file "10.10.200.inaddr.arpa"; };

cd /var/lib/bind/etc/zone
cp empty au-team.irpo
cp empty 10.10.100.in-addr.arpa
cp empty 10.10.200.in-addr.arpa

Прямая зона:

mcedit /var/lib/bind/etc/zone/au-team.irpo
zone$TTL 1D
@ IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
2026051001 12H 1H 1W 1H )
@ IN NS hq-srv.au-team.irpo.
hq-rtr IN A 10.10.100.1
br-rtr IN A 10.20.20.1
hq-srv IN A 10.10.100.2
hq-cli IN A 10.10.200.2
br-srv IN A 10.20.20.2
docker IN A 172.16.10.1
web IN A 172.16.20.1

Обратные зоны:

mcedit /var/lib/bind/etc/zone/10.10.100.in-addr.arpa
@ IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
2026051001 12H 1H 1W 1H )
@ IN NS hq-srv.au-team.irpo.
1 IN PTR hq-rtr.au-team.irpo.
2 IN PTR hq-srv.au-team.irpo.

mcedit /var/lib/bind/etc/zone/10.10.200.in-addr.arpa
@ IN SOA hq-srv.au-team.irpo. root.au-team.irpo. (
2026051001 12H 1H 1W 1H )
@ IN NS hq-srv.au-team.irpo.
1 IN PTR hq-rtr.au-team.irpo.
2 IN PTR hq-cli.au-team.irpo.

rndc-confgen > /etc/bind/rndc.key
sed -i '6,$d' /etc/bind/rndc.key
chown -R named:named /var/lib/bind/etc/zone/*
named-checkconf -z
systemctl enable --now bind
timedatectl set-timezone Europe/Moscow

```




 </details>
