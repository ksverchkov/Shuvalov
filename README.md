## Настройка топологии сетевой безопасности

Данная инструкция описывает настройку топологии сетевой безопасности с использованием следующих технологий:

- NAT (Network Address Translation)
- Фильтрация трафика
- Анализ входящего трафика
- ACL списки
- Firewall
- Reverse Proxy

### Хостнеймы:

1. **Gateway**: Имя хоста - `firewall-gw`
2. **Web Server**: Имя хоста - `web-server`
3. **Intrusion Detection System**: Имя хоста - `ids-server`

### Шаг 1: Настройка Gateway (firewall-gw)

#### 1.1 Настройка сетевого интерфейса

Откройте терминал на Gateway и выполните следующие команды:

```bash
sudo nano /etc/network/interfaces
```

Добавьте следующую конфигурацию для сетевого интерфейса eth0 (внешний интерфейс):

```plaintext
auto eth0
iface eth0 inet dhcp
```

Добавьте следующую конфигурацию для сетевого интерфейса eth1 (внутренний интерфейс):

```plaintext
auto eth1
iface eth1 inet static
    address 192.168.1.1
    netmask 255.255.255.0
```

Сохраните и закройте файл.

#### 1.2 Включение IP-маршрутизации

```bash
sudo nano /etc/sysctl.conf
```

Раскомментируйте или добавьте следующую строку:

```plaintext
net.ipv4.ip_forward=1
```

Сохраните и закройте файл.

#### 1.3 Перезапуск сетевых служб и IP-маршрутизации

```bash
sudo systemctl restart networking
sudo sysctl -p
```

### Шаг 2: Настройка Web Server (web-server)

#### 2.1 Настройка сетевого интерфейса

Откройте терминал на Web Server и выполните следующие команды:

```bash
sudo nano /etc/network/interfaces
```

Добавьте следующую конфигурацию для сетевого интерфейса eth0:

```plaintext
auto eth0
iface eth0 inet static
    address 192.168.1.2
    netmask 255.255.255.0
    gateway 192.168.1.1
```

Сохраните и закройте файл.

### Шаг 3: Настройка Intrusion Detection System (ids-server)

#### 3.1 Настройка сетевого интерфейса

Откройте терминал на Intrusion Detection System и выполните следующие команды:

```bash
sudo nano /etc/network/interfaces
```

Добавьте следующую конфигурацию для сетевого интерфейса eth0:

```plaintext
auto eth0
iface eth0 inet static
    address 

192.168.1.3
    netmask 255.255.255.0
    gateway 192.168.1.1
```

Сохраните и закройте файл.

### Дополнительная конфигурация

#### Настройка NAT на Gateway (firewall-gw)

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

#### Настройка ACL списков на Gateway (firewall-gw)

```bash
sudo iptables -A FORWARD -i eth1 -o eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state ESTABLISHED -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

#### Настройка Firewall на Gateway (firewall-gw)

```bash
sudo iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
sudo iptables -A INPUT -i eth0 -j DROP
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```

#### Настройка Reverse Proxy на Gateway (firewall-gw)

```bash
sudo apt update
sudo apt install nginx
sudo nano /etc/nginx/sites-available/reverse-proxy
```

Добавьте следующую конфигурацию в файл `/etc/nginx/sites-available/reverse-proxy`:

```plaintext
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://192.168.1.2:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Активируйте конфигурацию:

```bash
sudo ln -s /etc/nginx/sites-available/reverse-proxy /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

Теперь у вас настроена топология сетевой безопасности с использованием технологий NAT, фильтрации трафика, анализа входящего трафика, ACL списков, Firewall и Reverse Proxy. Обратите внимание, что эта инструкция представляет лишь основу, и в зависимости от ваших требований и конфигурации, может потребоваться дополнительная настройка и реализация механизмов безопасности.
