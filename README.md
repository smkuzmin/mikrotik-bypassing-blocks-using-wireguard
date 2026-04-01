## MikroTik: Обход блокировок используя Wireguard на зарубежном хостинге

Чтобы запустить Wireguard на MikroTik, обновите его прошивку до RouterOS 7.x.

Допустим, Wireguard на зарубежном хостинге (например: [HostVDS](https://hostvds.com/?affiliate_uuid=45e30dfa-ebf0-4f0c-aca2-3f3cfd07f84f), [VDSina](https://vdsina.com/ru)) выдал нам вот такой конфиг:

```ini
[Interface]
PrivateKey = PrIvaTeKeYxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
Address = 10.8.0.7/24
DNS = 8.8.8.8,1.1.1.1

[Peer]
PublicKey = PuBlIcKeYxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
PresharedKey = PrEsHaReDkEyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
Endpoint = 193.188.21.123:12674
```


Тогда настройка MikroTik будет выглядеть следующим образом:


1. **Создайте интерфейс Wireguard на RouterOS**

  Значение параметра **private-key** измените на соответствующие из полученного конфига Wireguard.

```ini
/interface wireguard
add name=wg1 private-key="PrIvaTeKeYxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx="
```


2. **Назначьте IP-адрес для интерфейса Wireguard**

  Здесь **10.8.0.7/24** - адрес из полученного конфига Wireguard.

```bash
/ip address
add address=10.8.0.7/24 interface=wg1
```


3. **Добавьте пир (узел) с параметрами из конфига**

  Все параметры измените на соответствующие из полученного конфига Wireguard.

```bash
/interface wireguard peers
add allowed-address=0.0.0.0/0,::/0 endpoint-address=193.188.21.123 endpoint-port=12674 interface=wg1 name=peer1 persistent-keepalive=25s preshared-key="PrEsHaReDkEyxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=" public-key="PuBlIcKeYxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx="
```


4. **Проверьте работу Wireguard**

```bash
/ping 8.8.8.8 interface=wg1
```


5. **Создайте списки частных сетей и сетей заблокированных сервисов**

```bash
/ip firewall address-list
remove [find dynamic=no list=PRIVATE-LANS]
add address=10.0.0.0/8     list=PRIVATE-LANS
add address=172.16.0.0/12  list=PRIVATE-LANS
add address=192.168.0.0/16 list=PRIVATE-LANS

/ip firewall address-list
remove [find dynamic=no list=CLOUDFLARE]
add address=8.6.112.0          list=CLOUDFLARE
add address=8.47.69.0          list=CLOUDFLARE
add address=35.190.80.1        list=CLOUDFLARE
add address=103.21.244.0/22    list=CLOUDFLARE
add address=103.22.200.0/22    list=CLOUDFLARE
add address=103.31.4.0/22      list=CLOUDFLARE
add address=104.16.0.0/13      list=CLOUDFLARE
add address=104.24.0.0/14      list=CLOUDFLARE
add address=108.162.192.0/18   list=CLOUDFLARE
add address=131.0.72.0/22      list=CLOUDFLARE
add address=141.101.64.0/18    list=CLOUDFLARE
add address=162.158.0.0/15     list=CLOUDFLARE
add address=172.64.0.0/13      list=CLOUDFLARE
add address=173.245.48.0/20    list=CLOUDFLARE
add address=188.114.96.0/20    list=CLOUDFLARE
add address=190.93.240.0/20    list=CLOUDFLARE
add address=197.234.240.0/22   list=CLOUDFLARE
add address=198.41.128.0/17    list=CLOUDFLARE

/ip firewall address-list
remove [find dynamic=no list=GOOGLEAI]
add address=64.233.161.95      list=GOOGLEAI
add address=64.233.162.95      list=GOOGLEAI
add address=64.233.163.95      list=GOOGLEAI
add address=64.233.164.95      list=GOOGLEAI
add address=64.233.165.95      list=GOOGLEAI
add address=74.125.131.95      list=GOOGLEAI
add address=74.125.205.95      list=GOOGLEAI
add address=108.177.14.95      list=GOOGLEAI
add address=142.250.150.95     list=GOOGLEAI
add address=142.251.1.95       list=GOOGLEAI
add address=172.253.130.95     list=GOOGLEAI
add address=172.253.152.95     list=GOOGLEAI
add address=173.194.73.95      list=GOOGLEAI
add address=173.194.220.95     list=GOOGLEAI
add address=173.194.221.95     list=GOOGLEAI
add address=173.194.222.95     list=GOOGLEAI
add address=209.85.233.95      list=GOOGLEAI
add address=216.239.32.223     list=GOOGLEAI
add address=216.239.34.223     list=GOOGLEAI
add address=216.239.36.223     list=GOOGLEAI
add address=216.239.38.223     list=GOOGLEAI

/ip firewall address-list
remove [find dynamic=no list=GOOGLEPLAY]
add address=34.104.35.123      list=GOOGLEPLAY
add address=46.61.154.76/31    list=GOOGLEPLAY
add address=46.61.154.78       list=GOOGLEPLAY
add address=46.61.154.80/31    list=GOOGLEPLAY
add address=46.61.154.83       list=GOOGLEPLAY
add address=64.233.161.94/31   list=GOOGLEPLAY
add address=64.233.161.100/31  list=GOOGLEPLAY
add address=64.233.161.102     list=GOOGLEPLAY
add address=64.233.161.113     list=GOOGLEPLAY
add address=64.233.161.119     list=GOOGLEPLAY
add address=64.233.161.132     list=GOOGLEPLAY
add address=64.233.161.138/31  list=GOOGLEPLAY
add address=64.233.162.94/31   list=GOOGLEPLAY
add address=64.233.162.100/31  list=GOOGLEPLAY
add address=64.233.162.102     list=GOOGLEPLAY
add address=64.233.162.113     list=GOOGLEPLAY
add address=64.233.162.119     list=GOOGLEPLAY
add address=64.233.162.132     list=GOOGLEPLAY
add address=64.233.162.138/31  list=GOOGLEPLAY
add address=64.233.163.94/31   list=GOOGLEPLAY
add address=64.233.163.100/31  list=GOOGLEPLAY
add address=64.233.163.102     list=GOOGLEPLAY
add address=64.233.163.113     list=GOOGLEPLAY
add address=64.233.163.119     list=GOOGLEPLAY
add address=64.233.163.138/31  list=GOOGLEPLAY
add address=64.233.164.94/31   list=GOOGLEPLAY
add address=64.233.164.100/31  list=GOOGLEPLAY
add address=64.233.164.102     list=GOOGLEPLAY
add address=64.233.164.113     list=GOOGLEPLAY
add address=64.233.164.119     list=GOOGLEPLAY
add address=64.233.164.132     list=GOOGLEPLAY
add address=64.233.164.138/31  list=GOOGLEPLAY
add address=64.233.165.94/31   list=GOOGLEPLAY
add address=64.233.165.100/31  list=GOOGLEPLAY
add address=64.233.165.102     list=GOOGLEPLAY
add address=64.233.165.113     list=GOOGLEPLAY
add address=64.233.165.119     list=GOOGLEPLAY
add address=64.233.165.132     list=GOOGLEPLAY
add address=64.233.165.138/31  list=GOOGLEPLAY
add address=74.125.8.1         list=GOOGLEPLAY
add address=74.125.8.2/31      list=GOOGLEPLAY
add address=74.125.8.4         list=GOOGLEPLAY
add address=74.125.8.38/31     list=GOOGLEPLAY
add address=74.125.8.40/31     list=GOOGLEPLAY
add address=74.125.8.42        list=GOOGLEPLAY
add address=74.125.8.70/31     list=GOOGLEPLAY
add address=74.125.8.72        list=GOOGLEPLAY
add address=74.125.8.74        list=GOOGLEPLAY
add address=74.125.8.102/31    list=GOOGLEPLAY
add address=74.125.8.104/31    list=GOOGLEPLAY
add address=74.125.8.166/31    list=GOOGLEPLAY
add address=74.125.8.168/31    list=GOOGLEPLAY
add address=74.125.8.170       list=GOOGLEPLAY
add address=74.125.8.198/31    list=GOOGLEPLAY
add address=74.125.8.200/31    list=GOOGLEPLAY
add address=74.125.8.202       list=GOOGLEPLAY
add address=74.125.8.230/31    list=GOOGLEPLAY
add address=74.125.8.232/31    list=GOOGLEPLAY
add address=74.125.8.234       list=GOOGLEPLAY
add address=74.125.11.9        list=GOOGLEPLAY
add address=74.125.11.10       list=GOOGLEPLAY
add address=74.125.11.36       list=GOOGLEPLAY
add address=74.125.11.102      list=GOOGLEPLAY
add address=74.125.11.105      list=GOOGLEPLAY
add address=74.125.13.230      list=GOOGLEPLAY
add address=74.125.13.232      list=GOOGLEPLAY
add address=74.125.99.97       list=GOOGLEPLAY
add address=74.125.99.99       list=GOOGLEPLAY
add address=74.125.99.101      list=GOOGLEPLAY
add address=74.125.100.7       list=GOOGLEPLAY
add address=74.125.100.8/31    list=GOOGLEPLAY
add address=74.125.100.10      list=GOOGLEPLAY
add address=74.125.100.38/31   list=GOOGLEPLAY
add address=74.125.100.41      list=GOOGLEPLAY
add address=74.125.100.42      list=GOOGLEPLAY
add address=74.125.100.70/31   list=GOOGLEPLAY
add address=74.125.100.74      list=GOOGLEPLAY
add address=74.125.100.102/31  list=GOOGLEPLAY
add address=74.125.100.104/31  list=GOOGLEPLAY
add address=74.125.100.106     list=GOOGLEPLAY
add address=74.125.100.134/31  list=GOOGLEPLAY
add address=74.125.100.136/31  list=GOOGLEPLAY
add address=74.125.100.138     list=GOOGLEPLAY
add address=74.125.100.161     list=GOOGLEPLAY
add address=74.125.100.162/31  list=GOOGLEPLAY
add address=74.125.100.164/31  list=GOOGLEPLAY
add address=74.125.100.198/31  list=GOOGLEPLAY
add address=74.125.100.200/31  list=GOOGLEPLAY
add address=74.125.100.230/31  list=GOOGLEPLAY
add address=74.125.100.232/31  list=GOOGLEPLAY
add address=74.125.100.234     list=GOOGLEPLAY
add address=74.125.104.74      list=GOOGLEPLAY
add address=74.125.104.97      list=GOOGLEPLAY
add address=74.125.110.102/31  list=GOOGLEPLAY
add address=74.125.110.105     list=GOOGLEPLAY
add address=74.125.110.106     list=GOOGLEPLAY
add address=74.125.111.134     list=GOOGLEPLAY
add address=74.125.111.137     list=GOOGLEPLAY
add address=74.125.111.138     list=GOOGLEPLAY
add address=74.125.111.166/31  list=GOOGLEPLAY
add address=74.125.111.168     list=GOOGLEPLAY
add address=74.125.131.94/31   list=GOOGLEPLAY
add address=74.125.131.100/31  list=GOOGLEPLAY
add address=74.125.131.102     list=GOOGLEPLAY
add address=74.125.131.113     list=GOOGLEPLAY
add address=74.125.131.119     list=GOOGLEPLAY
add address=74.125.131.132     list=GOOGLEPLAY
add address=74.125.131.138/31  list=GOOGLEPLAY
add address=74.125.153.7       list=GOOGLEPLAY
add address=74.125.153.8/31    list=GOOGLEPLAY
add address=74.125.153.198/31  list=GOOGLEPLAY
add address=74.125.153.201     list=GOOGLEPLAY
add address=74.125.153.202     list=GOOGLEPLAY
add address=74.125.154.71      list=GOOGLEPLAY
add address=74.125.154.72      list=GOOGLEPLAY
add address=74.125.154.74      list=GOOGLEPLAY
add address=74.125.154.103     list=GOOGLEPLAY
add address=74.125.154.104     list=GOOGLEPLAY
add address=74.125.154.106     list=GOOGLEPLAY
add address=74.125.154.134/31  list=GOOGLEPLAY
add address=74.125.154.137     list=GOOGLEPLAY
add address=74.125.154.138     list=GOOGLEPLAY
add address=74.125.160.38      list=GOOGLEPLAY
add address=74.125.160.198     list=GOOGLEPLAY
add address=74.125.160.201     list=GOOGLEPLAY
add address=74.125.160.202     list=GOOGLEPLAY
add address=74.125.160.231     list=GOOGLEPLAY
add address=74.125.160.232/31  list=GOOGLEPLAY
add address=74.125.160.234     list=GOOGLEPLAY
add address=74.125.162.7       list=GOOGLEPLAY
add address=74.125.162.38/31   list=GOOGLEPLAY
add address=74.125.162.74      list=GOOGLEPLAY
add address=74.125.162.102/31  list=GOOGLEPLAY
add address=74.125.162.104/31  list=GOOGLEPLAY
add address=74.125.162.134     list=GOOGLEPLAY
add address=74.125.162.136     list=GOOGLEPLAY
add address=74.125.162.138     list=GOOGLEPLAY
add address=74.125.162.167     list=GOOGLEPLAY
add address=74.125.162.168/31  list=GOOGLEPLAY
add address=74.125.162.193     list=GOOGLEPLAY
add address=74.125.162.195     list=GOOGLEPLAY
add address=74.125.162.197     list=GOOGLEPLAY
add address=74.125.163.137     list=GOOGLEPLAY
add address=74.125.163.167     list=GOOGLEPLAY
add address=74.125.163.231     list=GOOGLEPLAY
add address=74.125.163.233     list=GOOGLEPLAY
add address=74.125.173.6/31    list=GOOGLEPLAY
add address=74.125.173.9       list=GOOGLEPLAY
add address=74.125.173.38      list=GOOGLEPLAY
add address=74.125.173.137     list=GOOGLEPLAY
add address=74.125.173.169     list=GOOGLEPLAY
add address=74.125.173.170     list=GOOGLEPLAY
add address=74.125.173.202     list=GOOGLEPLAY
add address=74.125.173.230     list=GOOGLEPLAY
add address=74.125.173.233     list=GOOGLEPLAY
add address=74.125.205.94/31   list=GOOGLEPLAY
add address=74.125.205.100/31  list=GOOGLEPLAY
add address=74.125.205.102     list=GOOGLEPLAY
add address=74.125.205.113     list=GOOGLEPLAY
add address=74.125.205.119     list=GOOGLEPLAY
add address=74.125.205.132     list=GOOGLEPLAY
add address=74.125.205.138/31  list=GOOGLEPLAY
add address=87.245.216.172/31  list=GOOGLEPLAY
add address=87.245.220.76/30   list=GOOGLEPLAY
add address=87.245.220.80      list=GOOGLEPLAY
add address=87.245.222.236/31  list=GOOGLEPLAY
add address=108.177.14.94/31   list=GOOGLEPLAY
add address=108.177.14.100/31  list=GOOGLEPLAY
add address=108.177.14.102     list=GOOGLEPLAY
add address=108.177.14.113     list=GOOGLEPLAY
add address=108.177.14.119     list=GOOGLEPLAY
add address=108.177.14.132     list=GOOGLEPLAY
add address=108.177.14.138/31  list=GOOGLEPLAY
add address=142.250.74.3       list=GOOGLEPLAY
add address=142.250.74.10      list=GOOGLEPLAY
add address=142.250.74.14      list=GOOGLEPLAY
add address=142.250.150.94/31  list=GOOGLEPLAY
add address=142.250.150.100/31 list=GOOGLEPLAY
add address=142.250.150.102    list=GOOGLEPLAY
add address=142.250.150.113    list=GOOGLEPLAY
add address=142.250.150.119    list=GOOGLEPLAY
add address=142.250.150.138/31 list=GOOGLEPLAY
add address=142.251.1.94/31    list=GOOGLEPLAY
add address=142.251.1.100/31   list=GOOGLEPLAY
add address=142.251.1.102      list=GOOGLEPLAY
add address=142.251.1.113      list=GOOGLEPLAY
add address=142.251.1.119      list=GOOGLEPLAY
add address=142.251.1.132      list=GOOGLEPLAY
add address=142.251.1.138/31   list=GOOGLEPLAY
add address=142.251.38.99      list=GOOGLEPLAY
add address=142.251.38.106     list=GOOGLEPLAY
add address=142.251.38.110     list=GOOGLEPLAY
add address=142.251.38.118     list=GOOGLEPLAY
add address=142.251.142.227    list=GOOGLEPLAY
add address=142.251.142.234    list=GOOGLEPLAY
add address=142.251.142.238    list=GOOGLEPLAY
add address=142.251.143.138    list=GOOGLEPLAY
add address=142.251.143.142    list=GOOGLEPLAY
add address=142.251.143.150    list=GOOGLEPLAY
add address=172.217.19.227     list=GOOGLEPLAY
add address=172.217.19.234     list=GOOGLEPLAY
add address=172.217.19.238     list=GOOGLEPLAY
add address=172.217.19.246     list=GOOGLEPLAY
add address=172.217.20.170     list=GOOGLEPLAY
add address=172.217.20.174     list=GOOGLEPLAY
add address=172.217.20.182     list=GOOGLEPLAY
add address=172.217.132.6      list=GOOGLEPLAY
add address=172.217.132.8/31   list=GOOGLEPLAY
add address=172.217.132.10     list=GOOGLEPLAY
add address=172.217.132.38/31  list=GOOGLEPLAY
add address=172.217.132.40     list=GOOGLEPLAY
add address=172.217.132.71     list=GOOGLEPLAY
add address=172.217.132.74     list=GOOGLEPLAY
add address=172.217.132.102/31 list=GOOGLEPLAY
add address=172.217.132.104/31 list=GOOGLEPLAY
add address=172.217.132.106    list=GOOGLEPLAY
add address=172.217.132.134/31 list=GOOGLEPLAY
add address=172.217.132.136    list=GOOGLEPLAY
add address=172.217.132.138    list=GOOGLEPLAY
add address=172.217.132.166    list=GOOGLEPLAY
add address=172.217.132.168/31 list=GOOGLEPLAY
add address=172.217.132.170    list=GOOGLEPLAY
add address=172.217.132.198/31 list=GOOGLEPLAY
add address=172.217.132.201    list=GOOGLEPLAY
add address=172.217.132.202    list=GOOGLEPLAY
add address=172.217.133.166/31 list=GOOGLEPLAY
add address=172.217.133.169    list=GOOGLEPLAY
add address=172.217.133.170    list=GOOGLEPLAY
add address=172.217.133.198/31 list=GOOGLEPLAY
add address=172.217.133.200/31 list=GOOGLEPLAY
add address=172.217.133.230/31 list=GOOGLEPLAY
add address=172.217.133.232/31 list=GOOGLEPLAY
add address=172.217.133.234    list=GOOGLEPLAY
add address=172.253.130.94/31  list=GOOGLEPLAY
add address=172.253.130.100/31 list=GOOGLEPLAY
add address=172.253.130.102    list=GOOGLEPLAY
add address=172.253.130.113    list=GOOGLEPLAY
add address=172.253.130.119    list=GOOGLEPLAY
add address=172.253.130.132    list=GOOGLEPLAY
add address=172.253.130.138/31 list=GOOGLEPLAY
add address=172.253.152.94/31  list=GOOGLEPLAY
add address=172.253.152.100/31 list=GOOGLEPLAY
add address=172.253.152.102    list=GOOGLEPLAY
add address=172.253.152.113    list=GOOGLEPLAY
add address=172.253.152.119    list=GOOGLEPLAY
add address=172.253.152.138/31 list=GOOGLEPLAY
add address=173.194.1.8        list=GOOGLEPLAY
add address=173.194.6.134/31   list=GOOGLEPLAY
add address=173.194.6.137      list=GOOGLEPLAY
add address=173.194.6.166/31   list=GOOGLEPLAY
add address=173.194.6.168      list=GOOGLEPLAY
add address=173.194.6.170      list=GOOGLEPLAY
add address=173.194.6.198/31   list=GOOGLEPLAY
add address=173.194.6.200/31   list=GOOGLEPLAY
add address=173.194.6.230/31   list=GOOGLEPLAY
add address=173.194.6.232      list=GOOGLEPLAY
add address=173.194.6.234      list=GOOGLEPLAY
add address=173.194.10.38/31   list=GOOGLEPLAY
add address=173.194.10.41      list=GOOGLEPLAY
add address=173.194.10.42      list=GOOGLEPLAY
add address=173.194.10.70      list=GOOGLEPLAY
add address=173.194.10.102     list=GOOGLEPLAY
add address=173.194.10.104     list=GOOGLEPLAY
add address=173.194.10.106     list=GOOGLEPLAY
add address=173.194.16.134     list=GOOGLEPLAY
add address=173.194.16.136     list=GOOGLEPLAY
add address=173.194.16.168     list=GOOGLEPLAY
add address=173.194.16.198     list=GOOGLEPLAY
add address=173.194.18.74      list=GOOGLEPLAY
add address=173.194.18.134     list=GOOGLEPLAY
add address=173.194.19.8/31    list=GOOGLEPLAY
add address=173.194.19.10      list=GOOGLEPLAY
add address=173.194.19.40      list=GOOGLEPLAY
add address=173.194.19.67      list=GOOGLEPLAY
add address=173.194.19.68/31   list=GOOGLEPLAY
add address=173.194.73.94/31   list=GOOGLEPLAY
add address=173.194.73.100/31  list=GOOGLEPLAY
add address=173.194.73.102     list=GOOGLEPLAY
add address=173.194.73.113     list=GOOGLEPLAY
add address=173.194.73.119     list=GOOGLEPLAY
add address=173.194.73.138/31  list=GOOGLEPLAY
add address=173.194.153.6/31   list=GOOGLEPLAY
add address=173.194.153.8      list=GOOGLEPLAY
add address=173.194.153.10     list=GOOGLEPLAY
add address=173.194.153.33     list=GOOGLEPLAY
add address=173.194.153.35     list=GOOGLEPLAY
add address=173.194.153.36/31  list=GOOGLEPLAY
add address=173.194.182.73     list=GOOGLEPLAY
add address=173.194.182.102/31 list=GOOGLEPLAY
add address=173.194.182.104    list=GOOGLEPLAY
add address=173.194.182.200    list=GOOGLEPLAY
add address=173.194.182.234    list=GOOGLEPLAY
add address=173.194.187.8      list=GOOGLEPLAY
add address=173.194.187.135    list=GOOGLEPLAY
add address=173.194.187.136    list=GOOGLEPLAY
add address=173.194.187.202    list=GOOGLEPLAY
add address=173.194.187.231    list=GOOGLEPLAY
add address=173.194.187.232/31 list=GOOGLEPLAY
add address=173.194.188.8/31   list=GOOGLEPLAY
add address=173.194.188.10     list=GOOGLEPLAY
add address=173.194.188.38/31  list=GOOGLEPLAY
add address=173.194.188.41     list=GOOGLEPLAY
add address=173.194.188.42     list=GOOGLEPLAY
add address=173.194.188.71     list=GOOGLEPLAY
add address=173.194.188.72     list=GOOGLEPLAY
add address=173.194.188.74     list=GOOGLEPLAY
add address=173.194.188.105    list=GOOGLEPLAY
add address=173.194.188.136    list=GOOGLEPLAY
add address=173.194.188.138    list=GOOGLEPLAY
add address=173.194.188.166    list=GOOGLEPLAY
add address=173.194.188.168    list=GOOGLEPLAY
add address=173.194.188.199    list=GOOGLEPLAY
add address=173.194.188.201    list=GOOGLEPLAY
add address=173.194.188.202    list=GOOGLEPLAY
add address=173.194.220.94/31  list=GOOGLEPLAY
add address=173.194.220.100/31 list=GOOGLEPLAY
add address=173.194.220.102    list=GOOGLEPLAY
add address=173.194.220.113    list=GOOGLEPLAY
add address=173.194.220.119    list=GOOGLEPLAY
add address=173.194.220.138/31 list=GOOGLEPLAY
add address=173.194.221.94/31  list=GOOGLEPLAY
add address=173.194.221.100/31 list=GOOGLEPLAY
add address=173.194.221.102    list=GOOGLEPLAY
add address=173.194.221.113    list=GOOGLEPLAY
add address=173.194.221.119    list=GOOGLEPLAY
add address=173.194.221.132    list=GOOGLEPLAY
add address=173.194.221.138/31 list=GOOGLEPLAY
add address=173.194.222.94/31  list=GOOGLEPLAY
add address=173.194.222.100/31 list=GOOGLEPLAY
add address=173.194.222.102    list=GOOGLEPLAY
add address=173.194.222.113    list=GOOGLEPLAY
add address=173.194.222.119    list=GOOGLEPLAY
add address=173.194.222.138/31 list=GOOGLEPLAY
add address=192.178.25.14      list=GOOGLEPLAY
add address=209.85.226.6/31    list=GOOGLEPLAY
add address=209.85.226.9       list=GOOGLEPLAY
add address=209.85.226.10      list=GOOGLEPLAY
add address=209.85.226.38/31   list=GOOGLEPLAY
add address=209.85.226.40/31   list=GOOGLEPLAY
add address=209.85.226.42      list=GOOGLEPLAY
add address=209.85.226.65      list=GOOGLEPLAY
add address=209.85.226.67      list=GOOGLEPLAY
add address=209.85.233.94/31   list=GOOGLEPLAY
add address=209.85.233.100/31  list=GOOGLEPLAY
add address=209.85.233.102     list=GOOGLEPLAY
add address=209.85.233.113     list=GOOGLEPLAY
add address=209.85.233.119     list=GOOGLEPLAY
add address=209.85.233.132     list=GOOGLEPLAY
add address=209.85.233.138/31  list=GOOGLEPLAY
add address=213.59.237.212     list=GOOGLEPLAY
add address=213.59.237.214/31  list=GOOGLEPLAY
add address=213.59.237.216     list=GOOGLEPLAY
add address=213.59.237.218     list=GOOGLEPLAY
add address=213.59.237.220     list=GOOGLEPLAY
add address=213.59.237.223     list=GOOGLEPLAY
add address=216.58.201.170     list=GOOGLEPLAY
add address=216.58.201.174     list=GOOGLEPLAY
add address=216.58.201.182     list=GOOGLEPLAY
add address=216.58.201.195     list=GOOGLEPLAY
add address=216.58.201.202     list=GOOGLEPLAY
add address=216.58.201.206     list=GOOGLEPLAY
add address=216.58.201.234     list=GOOGLEPLAY
add address=216.58.201.238     list=GOOGLEPLAY
add address=216.239.32.223     list=GOOGLEPLAY
add address=216.239.34.223     list=GOOGLEPLAY
add address=216.239.36.223     list=GOOGLEPLAY
add address=216.239.38.223     list=GOOGLEPLAY

/ip firewall address-list
remove [find dynamic=no list=META]
add address=3.33.221.48        list=META
add address=3.33.252.61        list=META
add address=15.197.206.217     list=META
add address=15.197.210.208     list=META
add address=31.13.24.0/21      list=META
add address=31.13.64.0/18      list=META
add address=34.192.181.12      list=META
add address=34.193.38.112      list=META
add address=34.194.71.217      list=META
add address=34.194.255.230     list=META
add address=45.64.40.0/22      list=META
add address=57.141.0.0/24      list=META
add address=57.141.2.0/24      list=META
add address=57.141.4.0/24      list=META
add address=57.141.6.0/24      list=META
add address=57.141.8.0/24      list=META
add address=57.141.10.0/24     list=META
add address=57.141.12.0/24     list=META
add address=57.144.0.0/14      list=META
add address=66.220.144.0/20    list=META
add address=69.63.176.0/20     list=META
add address=69.171.224.0/19    list=META
add address=74.119.76.0/22     list=META
add address=102.132.96.0/20    list=META
add address=103.4.96.0/22      list=META
add address=129.134.0.0/17     list=META
add address=157.240.0.0/17     list=META
add address=157.240.192.0/18   list=META
add address=163.70.128.0/17    list=META
add address=173.252.64.0/18    list=META
add address=179.60.192.0/22    list=META
add address=185.60.216.0/22    list=META
add address=185.89.216.0/22    list=META
add address=204.15.20.0/22     list=META

/ip firewall address-list
remove [find dynamic=no list=TELEGRAM]
add address=91.105.192.0/23    list=TELEGRAM
add address=91.108.4.0/22      list=TELEGRAM
add address=91.108.8.0/21      list=TELEGRAM
add address=91.108.16.0/21     list=TELEGRAM
add address=91.108.56.0/22     list=TELEGRAM
add address=95.161.64.0/20     list=TELEGRAM
add address=149.154.160.0/20   list=TELEGRAM
add address=185.76.151.0/24    list=TELEGRAM

/ip firewall address-list
remove [find dynamic=no list=TORRENTS]
add address=3.135.72.151       list=TORRENTS
add address=18.219.255.217     list=TORRENTS
add address=37.221.67.160      list=TORRENTS
add address=104.21.95.93       list=TORRENTS
add address=168.119.95.238     list=TORRENTS
add address=172.67.144.20      list=TORRENTS
add address=193.46.255.29      list=TORRENTS

/ip firewall address-list
remove [find dynamic=no list=TWITTER]
add address=69.195.0.0/16      list=TWITTER
add address=104.16.0.0/12      list=TWITTER
add address=104.244.0.0/15     list=TWITTER
add address=107.167.27.0/24    list=TWITTER
add address=138.201.219.0/24   list=TWITTER
add address=146.75.0.0/16      list=TWITTER
add address=151.101.0.0/16     list=TWITTER
add address=162.158.0.0/15     list=TWITTER
add address=172.64.0.0/13      list=TWITTER
add address=185.45.5.0/24      list=TWITTER
add address=185.45.6.0/23      list=TWITTER
add address=185.199.0.0/16     list=TWITTER
add address=188.114.88.0/21    list=TWITTER
add address=188.186.0.0/16     list=TWITTER
add address=192.133.0.0/16     list=TWITTER
add address=199.16.0.0/13      list=TWITTER
add address=199.59.0.0/16      list=TWITTER
add address=199.96.56.0/23     list=TWITTER
add address=199.232.0.0/16     list=TWITTER
add address=209.237.0.0/16     list=TWITTER

/ip firewall address-list
remove [find dynamic=no list=VIBER]
add address=3.67.81.0/27       list=VIBER
add address=3.83.208.161       list=VIBER
add address=3.86.37.41         list=VIBER
add address=3.101.175.128/27   list=VIBER
add address=3.105.5.64/29      list=VIBER
add address=3.112.85.80/29     list=VIBER
add address=3.167.227.76       list=VIBER
add address=3.167.227.95       list=VIBER
add address=3.167.227.97       list=VIBER
add address=3.167.227.109      list=VIBER
add address=3.209.202.16/28    list=VIBER
add address=18.66.112.18       list=VIBER
add address=18.66.112.23       list=VIBER
add address=18.66.112.83       list=VIBER
add address=18.66.112.87       list=VIBER
add address=18.165.122.33      list=VIBER
add address=18.165.122.36      list=VIBER
add address=18.165.122.45      list=VIBER
add address=18.165.122.98      list=VIBER
add address=18.195.4.0/23      list=VIBER
add address=18.201.0.0/16      list=VIBER
add address=18.232.24.4        list=VIBER
add address=23.21.13.213       list=VIBER
add address=23.21.92.160       list=VIBER
add address=23.211.65.127      list=VIBER
add address=32.193.113.219     list=VIBER
add address=34.192.170.47      list=VIBER
add address=34.197.129.171     list=VIBER
add address=35.169.174.138     list=VIBER
add address=44.192.201.128/25  list=VIBER
add address=44.214.47.60       list=VIBER
add address=44.218.5.50        list=VIBER
add address=44.223.245.71      list=VIBER
add address=52.0.252.0/22      list=VIBER
add address=52.54.103.35       list=VIBER
add address=52.87.17.134       list=VIBER
add address=52.206.190.175     list=VIBER
add address=52.222.136.49      list=VIBER
add address=52.222.136.52      list=VIBER
add address=52.222.136.97      list=VIBER
add address=52.222.136.122     list=VIBER
add address=52.222.214.31      list=VIBER
add address=52.222.214.46      list=VIBER
add address=52.222.214.101     list=VIBER
add address=52.222.214.104     list=VIBER
add address=54.81.171.180      list=VIBER
add address=54.145.76.214      list=VIBER
add address=54.156.174.52      list=VIBER
add address=54.236.198.7       list=VIBER
add address=65.9.46.7          list=VIBER
add address=65.9.46.34         list=VIBER
add address=65.9.46.51         list=VIBER
add address=65.9.46.72         list=VIBER
add address=65.9.46.76         list=VIBER
add address=65.9.46.82         list=VIBER
add address=65.9.46.87         list=VIBER
add address=65.9.46.88         list=VIBER
add address=65.9.46.91         list=VIBER
add address=65.9.46.107        list=VIBER
add address=65.9.46.128        list=VIBER
add address=65.9.175.0/24      list=VIBER
add address=100.51.234.157     list=VIBER
add address=108.138.26.14      list=VIBER
add address=108.138.26.33      list=VIBER
add address=108.138.26.115     list=VIBER
add address=108.138.26.125     list=VIBER
add address=108.156.22.45      list=VIBER
add address=108.156.22.57      list=VIBER
add address=108.156.22.68      list=VIBER
add address=108.156.22.97      list=VIBER
add address=108.157.214.23     list=VIBER
add address=108.157.214.69     list=VIBER
add address=108.157.214.79     list=VIBER
add address=108.157.214.84     list=VIBER
add address=177.71.100.0/22    list=VIBER
add address=185.117.96.0/24    list=VIBER

/ip firewall address-list
remove [find dynamic=no list=YOUTUBE]
add address=5.143.239.205      list=YOUTUBE
add address=8.8.4.0/24         list=YOUTUBE
add address=8.8.8.0/24         list=YOUTUBE
add address=8.34.208.0/20      list=YOUTUBE
add address=8.35.192.0/20      list=YOUTUBE
add address=23.236.48.0/20     list=YOUTUBE
add address=23.251.128.0/19    list=YOUTUBE
add address=34.0.0.0/9         list=YOUTUBE
add address=34.128.0.0/10      list=YOUTUBE
add address=35.184.0.0/13      list=YOUTUBE
add address=35.192.0.0/14      list=YOUTUBE
add address=35.196.0.0/15      list=YOUTUBE
add address=35.198.0.0/16      list=YOUTUBE
add address=35.199.0.0/17      list=YOUTUBE
add address=35.199.128.0/18    list=YOUTUBE
add address=35.200.0.0/13      list=YOUTUBE
add address=35.208.0.0/12      list=YOUTUBE
add address=46.61.136.172      list=YOUTUBE
add address=46.61.154.76/30    list=YOUTUBE
add address=46.61.154.80/30    list=YOUTUBE
add address=46.61.170.76/31    list=YOUTUBE
add address=46.61.170.79       list=YOUTUBE
add address=46.61.170.80/30    list=YOUTUBE
add address=46.61.216.108      list=YOUTUBE
add address=46.61.216.172/31   list=YOUTUBE
add address=46.61.216.204/31   list=YOUTUBE
add address=64.18.0.0/15       list=YOUTUBE
add address=64.233.128.0/18    list=YOUTUBE
add address=66.102.0.0/20      list=YOUTUBE
add address=66.249.64.0/18     list=YOUTUBE
add address=70.32.128.0/19     list=YOUTUBE
add address=72.14.192.0/18     list=YOUTUBE
add address=74.114.24.0/21     list=YOUTUBE
add address=74.125.0.0/16      list=YOUTUBE
add address=80.252.155.12      list=YOUTUBE
add address=83.174.196.210     list=YOUTUBE
add address=83.174.199.111     list=YOUTUBE
add address=85.234.4.108/31    list=YOUTUBE
add address=87.245.216.172/31  list=YOUTUBE
add address=87.245.220.76/30   list=YOUTUBE
add address=87.245.220.80      list=YOUTUBE
add address=87.245.221.46      list=YOUTUBE
add address=87.245.222.236/31  list=YOUTUBE
add address=95.167.73.13       list=YOUTUBE
add address=95.167.73.44/31    list=YOUTUBE
add address=95.167.73.46       list=YOUTUBE
add address=95.167.73.108      list=YOUTUBE
add address=104.132.0.0/14     list=YOUTUBE
add address=104.152.0.0/14     list=YOUTUBE
add address=104.156.64.0/18    list=YOUTUBE
add address=104.196.0.0/14     list=YOUTUBE
add address=104.237.160.0/19   list=YOUTUBE
add address=107.167.160.0/19   list=YOUTUBE
add address=108.59.80.0/20     list=YOUTUBE
add address=108.170.192.0/18   list=YOUTUBE
add address=108.176.0.0/15     list=YOUTUBE
add address=109.195.25.140     list=YOUTUBE
add address=130.211.0.0/16     list=YOUTUBE
add address=136.112.0.0/12     list=YOUTUBE
add address=142.248.0.0/14     list=YOUTUBE
add address=146.148.0.0/14     list=YOUTUBE
add address=162.216.144.0/21   list=YOUTUBE
add address=162.222.176.0/21   list=YOUTUBE
add address=172.110.32.0/21    list=YOUTUBE
add address=172.217.0.0/16     list=YOUTUBE
add address=172.253.0.0/16     list=YOUTUBE
add address=173.194.0.0/16     list=YOUTUBE
add address=173.255.112.0/20   list=YOUTUBE
add address=178.66.83.172      list=YOUTUBE
add address=185.38.0.76/31     list=YOUTUBE
add address=188.43.61.140      list=YOUTUBE
add address=188.43.87.12       list=YOUTUBE
add address=192.158.28.0/22    list=YOUTUBE
add address=192.178.0.0/15     list=YOUTUBE
add address=193.186.4.0/24     list=YOUTUBE
add address=195.68.132.76      list=YOUTUBE
add address=199.36.154.0/23    list=YOUTUBE
add address=199.36.156.0/24    list=YOUTUBE
add address=199.192.112.0/21   list=YOUTUBE
add address=199.223.232.0/21   list=YOUTUBE
add address=207.126.144.0/20   list=YOUTUBE
add address=207.223.160.0/20   list=YOUTUBE
add address=208.65.152.0/22    list=YOUTUBE
add address=208.68.108.0/22    list=YOUTUBE
add address=208.81.188.0/22    list=YOUTUBE
add address=208.117.224.0/19   list=YOUTUBE
add address=209.85.128.0/17    list=YOUTUBE
add address=212.188.32.0/21    list=YOUTUBE
add address=213.59.210.16      list=YOUTUBE
add address=213.59.237.148/30  list=YOUTUBE
add address=213.59.237.153     list=YOUTUBE
add address=213.59.237.154/31  list=YOUTUBE
add address=213.59.237.158/31  list=YOUTUBE
add address=213.59.237.161     list=YOUTUBE
add address=213.59.237.163     list=YOUTUBE
add address=213.59.237.212/30  list=YOUTUBE
add address=213.59.237.216/31  list=YOUTUBE
add address=213.59.237.218     list=YOUTUBE
add address=213.59.237.220/30  list=YOUTUBE
add address=213.59.237.225     list=YOUTUBE
add address=213.59.237.226/31  list=YOUTUBE
add address=216.58.192.0/19    list=YOUTUBE
add address=216.239.32.0/19    list=YOUTUBE
add address=217.118.183.12     list=YOUTUBE
```


6. **Создайте правила для доменов заблокированных сервисов**

  Эти правила будут направлять нужные нам запросы вместе с поддоменами на DNS **77.88.8.88**, а полученные ответы автоматически заносить в соответствующие адрес-листы, где они будут жить до протухания кэша DNS.

  Вместо DNS от Яндекса (**77.88.8.88**) можно использовать любой симпатичный вам DNS, например от Cloudflare: **1.1.1.1**.

```bash
/ip dns static
remove [find address-list=CLOUDFLARE]
add address-list=CLOUDFLARE forward-to=77.88.8.88 match-subdomain=yes name=cdnjs.com            type=FWD
add address-list=CLOUDFLARE forward-to=77.88.8.88 match-subdomain=yes name=cloudflare.com       type=FWD
add address-list=CLOUDFLARE forward-to=77.88.8.88 match-subdomain=yes name=cloudflarestatus.com type=FWD
add address-list=CLOUDFLARE forward-to=77.88.8.88 match-subdomain=yes name=one.one.one.one      type=FWD

/ip dns static
remove [find address-list=GOOGLEAI]
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=ai.google.dev                      type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=aisandbox-pa.googleapis.com        type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=aistudio.google.com                type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=bard.google.com                    type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=clients6.google.com                type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=deepmind.com                       type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=deepmind.google                    type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=geller-pa.googleapis.com           type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=gemini.google                      type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=gemini.google.com                  type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=generativeai.google                type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=generativelanguage.googleapis.com  type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=jules.google                       type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=jules.google.com                   type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=labs.google                        type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=makersuite.google.com              type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=notebooklm.google                  type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=notebooklm.google.com              type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=proactivebackend-pa.googleapis.com type=FWD
add address-list=GOOGLEAI forward-to=77.88.8.88 match-subdomain=yes name=stitch.withgoogle.com              type=FWD

/ip dns static
remove [find address-list=GOOGLEPLAY]
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=android.clients.google.com                        type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=beacons.gvt2.com                                  type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=connectivitycheck.gstatic.com                     type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=googleplay.com                                    type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=gvt1.com                                          type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=lh3.googleusercontent.com                         type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=play-fe.googleapis.com                            type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=play-games.googleusercontent.com                  type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=play.google.com                                   type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=play.googleapis.com                               type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=play-lh.googleusercontent.com                     type=FWD
add address-list=GOOGLEPLAY forward-to=77.88.8.88 match-subdomain=yes name=prod-lt-playstoregatewayadapter-pa.googleapis.com type=FWD

/ip dns static
remove [find address-list=META]
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=cdninstagram.com type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=facebook.com     type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=facebook.net     type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=fb.com           type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=fbcdn.net        type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=fbsbx.com        type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=ig.me            type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=instagram.com    type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=internalfb.com   type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=meta.com         type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=oculus.com       type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=threads.net      type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=wa.me            type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=whatsapp.biz     type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=whatsapp.com     type=FWD
add address-list=META forward-to=77.88.8.88 match-subdomain=yes name=whatsapp.net     type=FWD

/ip dns static
remove [find address-list=TELEGRAM]
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=cdn-telegram.org type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=comments.app     type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=contest.com      type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=fragment.com     type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=graph.org        type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=quiz.directory   type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=t.me             type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=tdesktop.com     type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telega.one       type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegra.ph       type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegram-cdn.org type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegram.dog     type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegram.me      type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegram.org     type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telegram.space   type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=telesco.pe       type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=tg.dev           type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=tx.me            type=FWD
add address-list=TELEGRAM forward-to=77.88.8.88 match-subdomain=yes name=usercontent.dev  type=FWD

/ip dns static
remove [find address-list=TORRENTS]
add address-list=TORRENTS forward-to=77.88.8.88 match-subdomain=yes name=anybt.eth.limo  type=FWD
add address-list=TORRENTS forward-to=77.88.8.88 match-subdomain=yes name=cdnbase.com     type=FWD
add address-list=TORRENTS forward-to=77.88.8.88 match-subdomain=yes name=fast-torrent.ru type=FWD
add address-list=TORRENTS forward-to=77.88.8.88 match-subdomain=yes name=nnmclub.to      type=FWD
add address-list=TORRENTS forward-to=77.88.8.88 match-subdomain=yes name=rutor.info      type=FWD

/ip dns static
remove [find address-list=TWITTER]
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=ads-twitter.com         type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=cms-twdigitalassets.com type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=periscope.tv            type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=pscp.tv                 type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=t.co                    type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=tellapart.com           type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=tweetdeck.com           type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twimg.com               type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitpic.com             type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitter.biz             type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitter.com             type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitter.jp              type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twittercommunity.com    type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitterflightschool.com type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitterinc.com          type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitteroauth.com        type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twitterstat.us          type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twtrdns.net             type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twttr.com               type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twttr.net               type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=twvid.com               type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=vine.co                 type=FWD
add address-list=TWITTER forward-to=77.88.8.88 match-subdomain=yes name=x.com                   type=FWD

/ip dns static
remove [find address-list=VIBER]
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=vb.me         type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viber-bot.com type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viber-dev.com type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viber.com     type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viber.net     type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viberapi.com  type=FWD
add address-list=VIBER forward-to=77.88.8.88 match-subdomain=yes name=viberdns.com  type=FWD

/ip dns static
remove [find address-list=YOUTUBE]
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=ggpht.com                            type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=googlevideo.com                      type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=jnn-pa.googleapis.com                type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=nhacmp3youtube.com                   type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=returnyoutubedislikeapi.com          type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=wide-youtube.l.google.com            type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtu.be                             type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtube.com                          type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtube-nocookie.com                 type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtube-ui.l.google.com              type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtube.googleapis.com               type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtubeembeddedplayer.googleapis.com type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtubei.googleapis.com              type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=youtubekids.com                      type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=yt-video-upload.l.google.com         type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=yt.be                                type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=yt3.googleusercontent.com            type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=ytimg.com                            type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=ytimg.l.google.com                   type=FWD
add address-list=YOUTUBE forward-to=77.88.8.88 match-subdomain=yes name=yting.com                            type=FWD
```


7. **Создайте таблицу маршрутизации для VPN**

```bash
/routing table
add fib name=bypass-vpn
```


8. **Создайте правила, маркирующее трафик к заблокированным сервисам**

  Первое правило не позволит локальным адресам улететь в туннель.

```bash
/ip firewall mangle
add action=accept          chain=prerouting comment="ACCEPT ALL FROM PRIVATE-LANS TO PRIVATE-LANS"                            dst-address-list=PRIVATE-LANS src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO CLOUDFLARE AS BYPASS-VPN-CONN" connection-mark=no-mark connection-state=new dst-address-list=CLOUDFLARE new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO GOOGLEAI AS BYPASS-VPN-CONN"   connection-mark=no-mark connection-state=new dst-address-list=GOOGLEAI   new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO GOOGLEPLAY AS BYPASS-VPN-CONN" connection-mark=no-mark connection-state=new dst-address-list=GOOGLEPLAY new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO META AS BYPASS-VPN-CONN"       connection-mark=no-mark connection-state=new dst-address-list=META       new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO TELEGRAM AS BYPASS-VPN-CONN"   connection-mark=no-mark connection-state=new dst-address-list=TELEGRAM   new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO TORRENTS AS BYPASS-VPN-CONN"   connection-mark=no-mark connection-state=new dst-address-list=TORRENTS   new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO TWITTER AS BYPASS-VPN-CONN"    connection-mark=no-mark connection-state=new dst-address-list=TWITTER    new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO VIBER AS BYPASS-VPN-CONN"      connection-mark=no-mark connection-state=new dst-address-list=VIBER      new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-connection chain=prerouting comment="MARK CONNECTIONS ALL FROM PRIVATE-LANS TO YOUTUBE AS BYPASS-VPN-CONN"    connection-mark=no-mark connection-state=new dst-address-list=YOUTUBE    new-connection-mark=bypass-vpn-conn passthrough=yes src-address-list=PRIVATE-LANS
add action=mark-routing    chain=prerouting comment="MARK ROUTING ALL FROM PRIVATE-LANS MARKED VPN-CONN AS BYPASS-VPN"        connection-mark=bypass-vpn-conn                                             new-routing-mark=bypass-vpn      passthrough=no  src-address-list=PRIVATE-LANS
```


9. **Создайте правило, которое согласует MTU с VPN-интерфейсом**

```bash
/ip firewall mangle
add action=change-mss chain=forward comment="CHANGE MSS FOR TCP SYN TO BYPASS-VPN-IF" new-mss=clamp-to-pmtu out-interface=wg1 passthrough=yes protocol=tcp tcp-flags=syn
```


10. **Создайте правила маскарадинга для VPN-трафика**

  Первое правило не позволит маскарадить трафик из локальных сетей к локальным сетям.

```bash
/ip firewall nat
add action=accept chain=srcnat comment="ACCEPT ALL FROM PRIVATE-LANS TO PRIVATE-LANS FOR VPN" dst-address-list=PRIVATE-LANS src-address-list=PRIVATE-LANS
add action=masquerade chain=srcnat comment="MASQ ALL FROM PRIVATE-LANS MARKED AS BYPASS-VPN --> BYPASS-VPN-IF" out-interface=wg1 routing-mark=bypass-vpn src-address-list=PRIVATE-LANS
```


11. **Настройте маршрутизацию VPN-трафика через интерфейс Wireguard**

```bash
/ip route
add dst-address=0.0.0.0/0 gateway=wg1 routing-table=bypass-vpn
```


12. **Проверьте маршрутизацию с клиентского ПК**

  Трассировка ya.ru после нашего роутера НЕ ДОЛЖНА проходить через хоп из сети 10.8.0.0/24:
```powershell
C:\>tracert ya.ru

Трассировка маршрута к ya.ru [5.255.255.242]
с максимальным числом прыжков 30:

  1     2 ms     1 ms     1 ms  router [10.255.253.1]
  2     5 ms     6 ms     6 ms  96.234.80-1.samtel.ru [80.234.96.1]
```

  А трассировка googlevideo.com после нашего роутера ДОЛЖНА проходить через хоп из сети 10.8.0.0/24:
```powershell
C:\>tracert googlevideo.com

Трассировка маршрута к googlevideo.com [142.251.1.106]
с максимальным числом прыжков 30:

  1     2 ms     1 ms     2 ms  router [10.255.253.1]
  2    75 ms    74 ms    74 ms  10.8.0.1
```

  Если все так - поздравляю, вы сломали чебурнет!


**Ссылки по теме:**

 - [Смотрим рекламу на Youtube в 4K](https://telegra.ph/Smotrim-reklamu-na-Youtube-v-4K-08-12)
 - [Настройка клиента Wireguard на Mikrotik RouterOS для подключения к VPS, VDS серверу или готовой конфигурации](https://kiberlis.ru/mikrotik-wireguard-client)
 - [Wireguard в Mikrotik](https://www.youtube.com/live/eRcrZkwd5IM)
 - [Как определить оптимальный размер MTU?](https://help.keenetic.com/hc/ru/articles/214470885-%D0%9A%D0%B0%D0%BA-%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B8%D1%82%D1%8C-%D0%BE%D0%BF%D1%82%D0%B8%D0%BC%D0%B0%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-%D1%80%D0%B0%D0%B7%D0%BC%D0%B5%D1%80-MTU)
 - [YouTube Blocked hosts in Russia](https://gist.github.com/MashinaMashina/58eed9128ade2c8ff50a84d523ae97d1#file-youtube-blocked-hosts-in-russia-md)
 - [Диапазон IP-адресов Instagram, Netflix, ChatGPT, Youtube, Twitter](https://rockblack.su/vpn/dopolnitelno/diapazon-ip-adresov)
