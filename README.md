# 13-03
13-03 «Защита сети» Васяева Ирина
# Задание 1
#### 1: sudo nmap -sA 192.168.100.1
При выполнении этой команды, Nmap отправляет ACK-пакеты для проверки открытых портов. Пример вывода для этой команды может выглядеть так:
```
$ sudo nmap -sA 192.168.100.1

Starting Nmap 7.80 ( https://nmap.org ) at 2024-07-20 10:00 UTC
Nmap scan report for 192.168.100.1
Host is up (0.0010s latency).
Not shown: 993 closed ports
PORT     STATE    SERVICE
22/tcp open     ssh
80/tcp open     http
443/tcp open     https
```
В логах Suricata это может быть зафиксировано как следующее:
```
[INFO] [packet] 192.168.0.50:54432 -> 192.168.100.1:80 ACK
```
Если система распознает подозрительную активность, Fail2Ban может добавить IP-адрес в черный список:
```
[Fail2Ban] 192.168.0.50 has been banned for 3600 seconds
```
#### 2: sudo nmap -sT 192.168.100.1
При использовании команды TCP-соединения, вывод мог бы выглядеть так:
```
$ sudo nmap -sT 192.168.100.1

Starting Nmap 7.80 ( https://nmap.org ) at 2024-07-20 10:01 UTC
Nmap scan report for 192.168.100.1
Host is up (0.0010s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
21/tcp open  ftp
53/tcp open  domain
80/tcp open  http
```
Suricata может зафиксировать наличие SYN-пакетов следующим образом:
```
[INFO] [packet] 192.168.0.50:55555 -> 192.168.100.1:21 TCP SYN
```
Если система настроена на обнаружение вариативных подключений, Fail2Ban может начать блокировку IP-адреса:
```
[Fail2Ban] Detected 192.168.0.50 with multiple connections; banning for 600 seconds
```
#### 3: sudo nmap -sS 192.168.100.1
При выполнении SYN-сканирования, результаты были бы такими:
```
$ sudo nmap -sS 192.168.100.1

Starting Nmap 7.80 ( https://nmap.org ) at 2024-07-20 10:02 UTC
Nmap scan report for 192.168.100.1
Host is up (0.0010s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
23/tcp open     telnet
80/tcp open     http
```
Suricata снова может быть задействована, фиксируя это как потенциальное сканирование портов:
```
[ALERT] Detects potential port scanning from 192.168.0.50
```
Система безопасности может автоматически занести IP-адрес в свой список заблокированных:
```
[Fail2Ban] IP 192.168.0.50 has been blocked due to suspicious activity.
```
#### 4: sudo nmap -sV 192.168.100.1
Эта команда позволяет узнать версии служб, работающих на открытых портах. Приведем пример:
```
$ sudo nmap -sV 192.168.100.1

Starting Nmap 7.80 ( https://nmap.org ) at 2024-07-20 10:03 UTC
Nmap scan report for 192.168.100.1
Host is up (0.0010s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE VERSION
22/tcp open     ssh     OpenSSH 7.9
80/tcp open     http    Apache 2.4.41
```
Записи, которые фиксирует Suricata, могут быть следующими:
```
[INFO] [packet] 192.168.0.50:60000 -> 192.168.100.1:22 TCP SYN
```
И в случае подозрительной активности, Fail2Ban мог бы сделать следующее:
```
[Fail2Ban] 192.168.0.50 has triggered a ban due to suspicious version scanning.
```
# Задание 2
#### 1. Подготовка к атаке
Первым делом необходимо создать два файла: users.txt и pass.txt. Файл users.txt должен содержать имена пользователей, а файл pass.txt — возможные пароли. 
- users.txt:
```
user1
user2
admin
root
```
- pass.txt:
```
123456
password
letmein
qwerty
```
Теперь можно запустить атаку с помощью команды Hydra:
```
hydra -L users.txt -P pass.txt <192.168.100.1> ssh
```
#### 2. Ожидаемые результаты и логирование
- Логи Suricata
Suricata, как IDS/IPS, будет отслеживать сетевой трафик и фиксировать подозрительную активность. В логах Suricata могут появиться события, связанные с многократными попытками входа:
```
[INFO] [http] 192.168.0.50:12345 -> 192.168.100.1:22 PREDICTABLE-PORT ATTACK
[ALERT] [ssh_brute_force] Brute force attack detected from 192.168.0.50
```
- Логи Fail2Ban
Fail2Ban будет следить за логами SSH и блокировать IP-адреса после определенного количества неудачных попыток входа. После настройки файла /etc/fail2ban/jail.conf с enabled = true, в логах Fail2Ban можно увидеть что-то вроде:
```
2024-07-20 10:15:20 fail2ban.actions: WARNING [ssh] Ban 192.168.0.50
2024-07-20 10:15:20 fail2ban.actions: NOTICE [ssh] Unban 192.168.0.50 after 600 seconds
```
