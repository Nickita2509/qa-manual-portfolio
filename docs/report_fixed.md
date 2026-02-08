## Part 1. Инструмент ipcalc

- **1.1. Сети и маски**
  - **Адрес сети 192.167.38.54/13:**
    - Network: 192.160.0.0/13
  - **Перевод масок:**
    - 255.255.255.0 -> /24, 11111111.11111111.11111111.00000000
    - /15 -> 255.254.0.0, 11111111.11111110.00000000.00000000
    - 11111111.11111111.11111111.11110000 -> 255.255.255.240, /28
  - **Минимальный и максимальный хост в сети 12.167.38.4:**
    - /8 -> HostMin 12.0.0.1, HostMax 12.255.255.254
    - 11111111.11111111.00000000.00000000 -> HostMin 12.167.0.1, HostMax 12.167.255.254
    - 255.255.254.0 -> HostMin 12.167.38.1, HostMax 12.167.39.254
    - /4 -> HostMin 0.0.0.1, HostMax 15.255.255.254

- **1.2. localhost**
  - 194.34.23.100 — нет (не входит в 127.0.0.0/8)
  - 127.0.0.2 — да (входит в 127.0.0.0/8)
  - 127.1.0.1 — да (входит в 127.0.0.0/8)
  - 128.0.0.1 — нет (не входит в 127.0.0.0/8)

- **1.3. Диапазоны и сегменты сетей**
  - **Публичные/частные IP:**
    - 10.0.0.45 — частный
    - 134.43.0.2 — публичный
    - 192.168.4.2 — частный
    - 172.20.250.4 — частный
    - 172.0.2.1 — публичный
    - 192.172.0.1 — публичный
    - 172.68.0.2 — публичный
    - 172.16.255.255 — частный
    - 10.10.10.10 — частный
    - 192.169.168.1 — публичный
  - **Возможные IP-шлюзы в сети 10.10.0.0/18:**
    - 10.0.0.1 — нет
    - 10.10.0.2 — да
    - 10.10.10.10 — да
    - 10.10.100.1 — нет
    - 10.10.1.255 — да

## Part 2. Статическая маршрутизация между двумя машинами

- **Вывод существующих интерфейсов ws1 и ws2:**
  - Скрин: вывод `ip a` на ws1
    - ![ws1_ip](../misc/screenshots/ws1_ip.png)
  - Скрин: вывод `ip a` на ws2
    - ![ws2_ip](../misc/screenshots/ws2_ip.png)

- **Описание сетевого интерфейса и настройка адресов:**
  - Скрин: `etc/netplan/00-installer-config.yaml` на ws1
    - ![ws1_netplan](../misc/screenshots/ws1_netplan1.png)
  - Скрин: `etc/netplan/00-installer-config.yaml` на ws2
    - ![ws2_netplan](../misc/screenshots/ws2_netplan1.png)

- **Применение настроек:**
  - Команда: `sudo netplan apply`
  - Скрин: вывод `netplan apply` на ws1
    - ![ws1_netplan_apply](../misc/screenshots/ws1-netplan_apply1.png)
  - Скрин: вывод `netplan apply` на ws2
    - ![ws2_netplan_apply](../misc/screenshots/ws2-netplan_apply1.png)

- **2.1. Добавление статического маршрута вручную**
  - Скрин: `sudo ip route add` на ws1
    - ![ws1_ip_add](../misc/screenshots/ws1-ip_add.png)
  - Скрин: `sudo ip route add` на ws2
    - ![ws2_ip_add](../misc/screenshots/ws2-ip_add.png)
  - Скрин: ping ws1 -> ws2
    - ![ws1_ping_ws2](../misc/screenshots/ping_ws1_to_ws2.png)
  - Скрин: ping ws2 -> ws1
    - ![ws2_ping_ws1](../misc/screenshots/ping_ws2_to_ws1.png)

- **2.2. Добавление статического маршрута с сохранением**
  - Скрин: `etc/netplan/00-installer-config.yaml` на ws1
    - ![ws1_netplan_ws2](../misc/screenshots/ws1-netplan-ws2.png)
  - Скрин: `etc/netplan/00-installer-config.yaml` на ws2
    - ![ws2_netplan_ws1](../misc/screenshots/ws2-netplan-ws1.png)
  - Скрин: ping ws1 -> ws2
    - ![ws1_ping_ws2_2](../misc/screenshots/ws1-ping-ws2.png)
  - Скрин: ping ws2 -> ws1
    - ![ws2_ping_ws1_2](../misc/screenshots/ws2-ping-ws1.png)

## Part 3. Утилита iperf3

- **3.1. Скорость соединения (перевод единиц):**
  - 8 Mbps -> 1 MB/s (делим на 8)
  - 100 MB/s -> 800 000 Kbps (100 MB/s * 8 = 800 Mbps; 800 Mbps * 1000 = 800 000 Kbps)
  - 1 Gbps -> 1000 Mbps

- **3.2. Утилита iperf3:**
  - Установка: `sudo apt install iperf3 -y`
  - Скрин: ws1 (server)
    - ![ws1-server](../misc/screenshots/speed-ws1-server.png)
  - Скрин: ws2 (client)
    - ![ws2-client](../misc/screenshots/speed-ws2-client.png)
  - Результат скорости: 10.0 Gbits/sec
  - Скрин: ws1 (client)
    - ![ws1-client](../misc/screenshots/speed-ws1-client.png)
  - Скрин: ws2 (server)
    - ![ws2-server](../misc/screenshots/speed-ws2-server.png)
  - Результат скорости: 9.83 Gbits/sec

## Part 4. Сетевой экран

- **4.1. Утилита iptables**
  - Создан файл `/etc/firewall.sh` на ws1 и ws2
  - Правила:
    - Очистка таблицы `filter`
    - Разрешение портов 22 (SSH) и 80 (HTTP)
    - На ws1: сначала DROP для echo-reply, затем ACCEPT
    - На ws2: сначала ACCEPT для echo-reply, затем DROP
  - Скрин: содержимое правил на ws1
    - ![ws1-filter](../misc/screenshots/ws1-filters.png)
  - Скрин: содержимое правил на ws2
    - ![ws2-filter](../misc/screenshots/ws2-filters.png)
  - Выполнены команды:
    - `sudo chmod +x /etc/firewall.sh`
    - `sudo /etc/firewall.sh`
    - `sudo iptables -L -n -v`
  - Скрин: вывод iptables на ws1
    - ![ws1-iptables](../misc/screenshots/ws1-iptables.png)
  - Скрин: вывод iptables на ws2
    - ![ws2-iptables](../misc/screenshots/ws2-iptables.png)
  - Разница стратегий:
    - ws1 (DROP -> ACCEPT): echo-reply блокируется, последующее ACCEPT не срабатывает
    - ws2 (ACCEPT -> DROP): echo-reply разрешается первым правилом, последующее DROP не влияет

- **4.2. Утилита nmap**
  - Скрин: ping ws1 -> ws2
    - ![ws1-ping-iptables](../misc/screenshots/ws1-ping-iptables.png)
  - Скрин: ping ws2 -> ws1
    - ![ws2-ping-iptables](../misc/screenshots/ws2-ping-iptables.png)
  - Результат: ws1 не пингуется с ws2
  - Скрин: `nmap` ws2 -> ws1
    - ![ws2-nmap](../misc/screenshots/ws2-nmap.png)
  - Результат: Host is up

## Part 5. Статическая маршрутизация сети

- **5.1. Настройка адресов машин**
  - Поднято 5 ВМ: ws11, ws21, ws22, r1, r2
  - Настроены статические IP в `/etc/netplan/01-static.yaml`
  - Скрин: ws11 netplan
    - ![ws11-netplan](../misc/screenshots/ws11_netplan.png)
  - Скрин: ws21 netplan
    - ![ws21-netplan](../misc/screenshots/ws21_netplan.png)
  - Скрин: ws22 netplan
    - ![ws22-netplan](../misc/screenshots/ws22_netplan.png)
  - Скрин: r1 netplan
    - ![r1-netplan](../misc/screenshots/r1_netplan.png)
  - Скрин: r2 netplan
    - ![r2-netplan](../misc/screenshots/r2_netplan.png)
  - Применение: `sudo netplan apply`
  - Проверка IP: `ip -4 a`
  - Скрин: ip a ws11
    - ![ip-a-ws11](../misc/screenshots/ws11_ipa.png)
  - Скрин: ip a ws21
    - ![ip-a-ws21](../misc/screenshots/ws21_ipa.png)
  - Скрин: ip a ws22
    - ![ip-a-ws22](../misc/screenshots/ws22_ipa.png)
  - Скрин: ip a r1
    - ![ip-a-r1](../misc/screenshots/r1_ipa.png)
  - Скрин: ip a r2
    - ![ip-a-r2](../misc/screenshots/r2_ipa.png)
  - Проверка связности:
    - ping ws22 с ws21
    - ping r1 с ws11
  - Скрин: ping ws21 -> ws22
    - ![ws21-ping-ws22](../misc/screenshots/ws21-ping-ws22.png)
  - Скрин: ping ws11 -> r1
    - ![ws11-ping-r1](../misc/screenshots/ws11-ping-r1.png)

- **5.2. Включение переадресации IP**
  - Временно: `sysctl -w net.ipv4.ip_forward=1`
  - Скрин: r1
    - ![ip-forward-temp](../misc/screenshots/r1_sys_ip_fwd.png)
  - Скрин: r2
    - ![ip-forward-temp2](../misc/screenshots/r2_sys_ip_fwd.png)
  - Постоянно: добавлена строка `net.ipv4.ip_forward = 1` в `/etc/sysctl.conf`
  - Скрин: sysctl.conf на r1
    - ![sysctl-conf](../misc/screenshots/r1_sysconf.png)
  - Скрин: применение sysctl на r1
    - ![sysctl-conf-p](../misc/screenshots/r1-sysctl.png)
  - Скрин: sysctl.conf на r2
    - ![sysctl-conf2](../misc/screenshots/r2_sysconf.png)
  - Скрин: применение sysctl на r2
    - ![sysctl-conf2-p](../misc/screenshots/r2_sysctl.png)

- **5.3. Маршрут по умолчанию**
  - Добавлен default route в `/etc/netplan/01-static.yaml`
  - Скрин: ws11 netplan
    - ![ws11-default-route-netplan](../misc/screenshots/ws11_netplan_r1.png)
  - Скрин: ws21 netplan
    - ![ws21-default-route-netplan](../misc/screenshots/ws21_netplan_r2.png)
  - Скрин: ws22 netplan
    - ![ws22-default-route-netplan](../misc/screenshots/ws22_netplan_r2.png)
  - Применение: `sudo netplan apply`
  - Проверка маршрутов: `ip r`
  - Скрин: ip r ws11
    - ![ws11-ip-r](../misc/screenshots/ws11_ipr.png)
  - Скрин: ip r ws21
    - ![ws21-ip-r](../misc/screenshots/ws21_ipr.png)
  - Скрин: ip r ws22
    - ![ws22-ip-r](../misc/screenshots/ws22_ipr.png)
  - Ping ws11 -> r2
    - ![ws11_ping_r2](../misc/screenshots/ws11_ping_r2.png)
  - tcpdump на r2: `sudo tcpdump -tn -i enp0s1 icmp`
    - ![r2-tcpdump-icmp](../misc/screenshots/r2_tcpdump.png)
  - Результат: в tcpdump видны ICMP echo-запросы от ws11

## Part 6. Динамическая настройка IP с помощью DHCP

- **Статус:** не выполнено

## Part 7. NAT

- **Статус:** не выполнено

## Part 8. Дополнительно. Знакомство с SSH Tunnels

- **Статус:** не выполнено
