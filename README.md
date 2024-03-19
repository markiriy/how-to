# how-to
![image](https://github.com/markiriy/how-to/assets/124806098/ca7024c1-aadb-48f0-9707-e317974d8094)


# БАЗА
sysctl -w net.ipv4.ip_forward=1 >> /etc/sysctl.conf
sysctl -p

# Routing
root@isp:~# nano /etc/network/interfaces
up ip route add 172.16.10.0/26 via 5.5.5.6
up ip route add 192.168.10.0/28 via 6.6.6.7

# GRE
br-r  gre            hq-r  gre
      gre1                 gre1
      ens192               ens192
      6.6.6.7              5.5.5.6
      5.5.5.6              6.6.6.7
      10.5.5.2/30          10.5.5.1/30
nmcli connection modify gre ip-tunnel.ttl 64

# IPSEC
apt install libreswan -y
hq: nano /etc/ipsec.d/wsr.conf
    conn wsr
      auto=start
      authby=secret
      type=tunnel
      ike=aes-sha1;modp2048
      left=10.5.5.1
      leftprotoport=gre
      right=10.5.5.2
      rightprotoport=gre
nano /etc/ipsec.d/wsr.secrets
10.5.5.1 10.5.5.2 : PSK "P@$$w0rd"

br: nano /etc/ipsec.d/wsr.conf
conn wsr
      auto=start
      authby=secret
      type=tunnel
      ike=aes-sha1;modp2048
      left=10.5.5.2
      leftprotoport=gre
      right=10.5.5.1
      rightprotoport=gre
nano /etc/ipsec.d/wsr.secrets
10.5.5.2 10.5.5.1 : PSK "P@$$w0rd"

# DHCP
![image](https://github.com/markiriy/how-to/assets/124806098/ddb15c86-78bc-4d6d-9706-1c4044c3f5cd)
![image](https://github.com/markiriy/how-to/assets/124806098/61338d70-7a95-4914-bb5d-dde2c3ab781e)
subnet 172.16.10.0 netmask 255.255.255.192 {
range 172.16.10.1 172.16.10.5;
option routers 172.16.10.1;
}
host hqsrv {
hardware ethernet mac-address;
 fixed-address 172.16.10.10;
}

# USERS
cli
adduser admin
usermod -c "Admin" admin

hq-srv
adduser admin

hq-r
adduser admin
adduser network_admin

br-srv
adduser branch_admin
adduser network_admin

br-r
adduser branch_admin
adduser network_admin

# FREEIPA
root@hq-srv:~# apt-get install freeipa-server freeipa-server-dns -y
root@hq-srv:~# ipa-server-install —-setup-dns —-allow-zone-overlap
https://hq-srv.hq.work/
BR-SRV & CLI
root@br-srv&&cli:~# apt-get install freeipa-client task-auth-freeipa -y
root@br-srv&&cli:~# ipa-client-install
in the end user authorized: admin P@ssw0rd

# SSH
ALT
echo Port 2222 >> /etc/openssh/sshd_config
echo Port 2222 >> /etc/openssh/ssh_config
echo "Authorized access only!" | tee /etc/issue.net
echo "Banner /etc/issue.net" >> /etc/openssh/sshd_config
echo "MaxAuthTries 4" >> /etc/openssh/sshd_config
echo "PermitEmptyPasswords no" >> /etc/openssh/sshd_config
echo "LoginGraceTime 300" >> /etc/openssh/sshd_config
echo "PermitRootLogin no" >> /etc/openssh/sshd_config

echo "PasswordAuthentication no" >> /etc/openssh/sshd_config

DEB
echo Port 2222 >> /etc/ssh/sshd_config
echo Port 2222 >> /etc/ssh/ssh_config
echo "Authorized access only!" | tee /etc/issue.net
echo "Banner /etc/issue.net" >> /etc/ssh/sshd_config
echo "MaxAuthTries 4" >> /etc/ssh/sshd_config
echo "PermitEmptyPasswords no" >> /etc/ssh/sshd_config
echo "LoginGraceTime 300" >> /etc/ssh/sshd_config
echo "PermitRootLogin no" >> /etc/ssh/sshd_config

echo "PasswordAuthentication no" >> /etc/ssh/sshd_config
root@hq-r:~# iptables -t nat -A PREROUTING -p tcp —-dport 22 -j DNAT —-to-destination 172.16.10.10:2222
root@hq-srv:~# vim /etc/hosts.deny
sshd:10.0.0.10
systemctl restart sshd.service
systemctl enable sshd.service

# BACKUP
For it you need root access via ssh (PermitRootLogin yes)
root@hq-srv:~# mkdir -p /backup/hq-r
root@hq-srv:~# mkdir /backup/br-r
root@hq-srv:~# vim backup
            #!/bin/bash
            scp -r root@hq-r:/etc/NetworkManager/system-connections /backup/hq-r
            scp -r root@br-r:/etc/NetworkManager/system-connections /backup/br-r
bash backup

# NTP
LOOPBACK HQR: 
nmcli connection add connection.id lo1 connection.interface-name lo1 connection.type dummy connection.autoconnect yes ipv4.method manual ipv4.address 1.1.1.1/32
chrony:
local stratum 5
manual
allow
EVERYONE:timedatectl set-timezone Europe/Moscow

# IPERF3
ISP
apt install iperf
iperf -c 5.5.5.6
HQ-R
apr install iperf
iperf -s

# RAID5
mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sdb /dev/sdc /dev/sdd

# IPTABLES
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -m state --state INVALID -j DROP
iptables -A INPUT -p tcp -m multiport --dports 53,80,443,22,2222 -j ACCEPT
iptables -A INPUT -p udp -m multiport --dports 53,500,4500 -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -p gre -j ACCEPT
iptables -A INPUT -i ens192 -j ACCEPT
iptables -A INPUT -i ens224 -j ACCEPT
iptables -A INPUT -i gre1 -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m state --state NEW -j ACCEPT
iptables -A FORWARD -m state --state INVALID -j DROP
iptables -A FORWARD -i ens192 -j ACCEPT
iptables -A FORWARD -i ens224 -j ACCEPT

# ClamAV
HQ-SRV и BR-SRV
apt-get install clamav
/etc/clamav/freshclam.conf
![image](https://github.com/markiriy/how-to/assets/124806098/baf5be48-58f2-402f-ab50-99e97511c458)

# CUPS
Необходимо на BR-SRV на рабочем столе создать обычный пустой файл
Открываем этот файл и жмем на печать
Выбираем параметр Cups-PDF и жмем печать. На рабочий стол сохранится файл в PDF.

# MOODLE
Открываем Synaptic, ставим moodle, moodle-apache2, moodle-base, moodle-local-mysql
root@br-srv:~# systemctl enable —-now httpd2 mariadb
Также необходимо отредактировать параметр max_input_vars поставив значение 6000 в файле:
root@br-srv:~# nano /etc/php/8.0/apache2-mod_php/php.ini max_input_vars 6000
Далее необходимо скачать сам moodle https://download.moodle.org/releases/latest
После этого распаковать архив.
Далее создаем директории для moodle:
root@br-srv:~# mkdir /var/www/moodledata
root@br-srv:~# chown -R apache2:apache2 /var/www/moodledata
root@br-srv:~# chmod -R 755 /var/www/moodledata
root@br-srv:~# cp -r /home/user/moodle /var/www/html/
Заходим на http://localhost/moodle 
Выбираем язык 
Проверяем, чтобы верно были указаны каталоги
Выбираем базу данных MariaDB и указываем параметры подключения к ней:
      Сервер БД: localhost
      Название БД: moodle
      Пользователь БД: root
      Пароль:
      Префикс: mdl_
      Порт: 3306
      Unix-сокет: /var/lib/mysql/mysql.sock
Принимаем условия 
Проверяем, что конфигурация сервера отвечает минимальным требованиям , если нет, то устраняем проблемы
Указываем обязательные данные:
      Логин: admin
      Пароль: P@ssw0rd
      Имя: Администратор
      Фамилия: Пользователь
      Эл.почта: admin@admin.conf
      Показывать:всем
Прописываем параметры главной страницы(абсолютно любые)
Любые контакты
Настройка подписи рабочего места в данном случае выполняется следующим способом. Переходим на домашнюю страницу сайта: Sitehome
Далее Turn editing on
Жмем на шестереночку, которая под надписью “test”
В поле Summary пишем номер рабочего места, сохраняем.
Далее необходимо создать группы и пользователей в соответствии с таблицей. Для этого переходим в раздел “Site administration”, “Пользователи”, “Учетные записи”, “Глобальные группы”
Жмем “Добавить глобальную группу” и создаем необходимые группы
Далее необходимо создать пользователей и присоединить его к соответствующей группе.
Для этого переходим в “Администрирование”, “Пользователи”, “Учетные записи”, “Добавить пользователя”
Далее необходимо добавить пользователей в соответствующие группы, для этого необходимо перейти снова к “Глобальные группы”, выбрать группу и нажать “Назначить” 
