# **Настройка firewalld для шлюза**

## **Содержание**
- [**Настройка firewalld для шлюза**](#настройка-firewalld-для-шлюза)
  - [**Содержание**](#содержание)
  - [**Диагностика**](#диагностика)
  - [**Определение зон**](#определение-зон)
  - [**Пример проброса 8080 на UTM (192.168.150.166:8080)**](#пример-проброса-8080-на-utm-1921681501668080)
  - [**Разрешение маршрутизации в trusted**](#разрешение-маршрутизации-в-trusted)
  - [**Настройка NAT для выхода в интернет**](#настройка-nat-для-выхода-в-интернет)
  - [**Разрешение SSH на WAN**](#разрешение-ssh-на-wan)
  - [**Проброс портов (forward) для 212.6.5.219 на 172.16.200.254**](#проброс-портов-forward-для-21265219-на-17216200254)
  - [**Проброс портов 80,443,22 с 212.6.5.212 на 172.16.200.5**](#проброс-портов-8044322-с-21265212-на-172162005)
  - [**Применение настроек**](#применение-настроек)
  - [**Дополнительные замечания**](#дополнительные-замечания)

## **Диагностика**

```bash
firewall-cmd --get-active-zones
# Пример вывода
# public
#   interfaces: enp1s0
# trusted
#   interfaces: wg+ tun+ enp2s0

firewall-cmd --zone=public --list-all
# Пример вывода
# public (active)
#   target: default
#   icmp-block-inversion: no
#   interfaces: enp1s0
#   sources:
#   services: dhcpv6-client ssh
#   ports:
#   protocols:
#   masquerade: yes
#   forward-ports: port=8080:proto=tcp:toport=8080:toaddr=192.168.150.166
#   source-ports:
#   icmp-blocks:
#   rich rules:

firewall-cmd --zone=trusted --list-all
# trusted (active)
#   target: ACCEPT
#   icmp-block-inversion: no
#   interfaces: enp2s0 tun+ wg+
#   sources:
#   services: dhcp dns ssh
#   ports:
#   protocols:
#   masquerade: no
#   forward-ports:
#   source-ports:
#   icmp-blocks:
#   rich rules:
#         rule family="ipv4" source address="192.168.150.0/24" masquerade
```

## **Определение зон**

Настроим интерфейсы в соответствующих зонах:

- `lan`: `enp1s0`, `tun+`, `wg+` → `trusted`
- `wan`: `enp2s0` → `public`

```bash
firewall-cmd --permanent --zone=trusted --add-interface=enp1s0
firewall-cmd --permanent --zone=trusted --add-interface=tun+
firewall-cmd --permanent --zone=trusted --add-interface=wg+
firewall-cmd --permanent --zone=public --add-interface=enp2s0
```

## **Пример проброса 8080 на UTM (192.168.150.166:8080)**

```bash
firewall-cmd --permanent --zone=public --add-forward-port=port=8080:proto=tcp:toport=8080:toaddr=192.168.150.166
# iptables -t nat -S|grep NAT|grep 192.168.150.166:8080
# Выведет:
# -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.150.166:8080
#-A FORWARD -p tcp -d 192.168.150.166 --dport 8080 -j ACCEPT


```

## **Разрешение маршрутизации в trusted**

```bash
firewall-cmd --permanent --zone=trusted --add-masquerade
firewall-cmd --permanent --zone=trusted --add-forward
# Проверить будет ли работать без правил ниже
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i enp1s0 -o tun+ -j ACCEPT
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i tun+ -o enp1s0 -j ACCEPT
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i enp1s0 -o wg+ -j ACCEPT
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i wg+ -o enp1s0 -j ACCEPT
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i tun+ -o wg+ -j ACCEPT
# firewall-cmd --permanent --direct --add-rule ipv4 filter FORWARD 0 -i wg+ -o tun+ -j ACCEPT
```

## **Настройка NAT для выхода в интернет**

```bash
firewall-cmd --permanent --zone=public --add-masquerade
# другой вариант с использованием iptables
# firewall-cmd --permanent --direct --add-rule ipv4 nat POSTROUTING 0 -o enp2s0 -j MASQUERADE
```

## **Разрешение SSH на WAN**

```bash
firewall-cmd --permanent --zone=public --add-service=ssh
```

## **Проброс портов (forward) для 212.6.5.219 на 172.16.200.254**
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="212.6.5.219" forward to address="172.16.200.254"'
```

## **Проброс портов 80,443,22 с 212.6.5.212 на 172.16.200.5**
```bash
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="212.6.5.212" port protocol="tcp" port="80" forward to address="172.16.200.5"'

# Проброс порта 443 для 212.6.5.212 на 172.16.200.5
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="212.6.5.212" port protocol="tcp" port="443" forward to address="172.16.200.5"'

# Проброс порта 22 для 212.6.5.212 на 172.16.200.5
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="212.6.5.212" port protocol="tcp" port="22" forward to address="172.16.200.5"'
```

## **Применение настроек**

```bash
firewall-cmd --reload
```

## **Дополнительные замечания**

- Если используется `external` зона вместо `public`, потребуется явно разрешить `forward`:
```bash
  firewall-cmd --permanent --zone=external --add-forward
```

После выполнения этих шагов шлюз будет работать корректно, обеспечивая NAT и маршрутизацию в `trusted` сети.

