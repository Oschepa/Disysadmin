2. Помощи не жди !!!! обратной пути нету !!!!

<h1>chrony (hq-rtr)</h1>

nano /etc/chrony.conf

![](/2_mod/1.jpg)

systemctl restart chronyd.service

<h1>на пользователях (br-rtr, hq-cli, hq-srv, br-srv) </h1>

nano /etc/chrony.conf

![](/2_mod/2.jpg)

systemctl restart chronyd.service <br>
для проверки chronyc sources

![](/2_mod/3.jpg)

<h1>raid массив (hq-srv)</h1>

mdadm –create /dev/md0 –level=5 –raid-devices=3 /dev/vdb  /dev/vdc /dev/vdd <br>
fdisk /dev/md0 (далее n и enter для сохранения w) <br>
mkfs.ext4 /dev/md0p1 <br>
mkdir /raid5

<h1>2 способа монтирования (uuid и директория)</h1>

uuid <br>
blkid /dev/md0p1 <br>
скопировать uuid <br>
nano /etc/fstab 

![](/2_mod/4.jpg)

<h2>директория</h2>

![](/2_mod/5.jpg)

mount -a на проверку ошибок <br>
systemctl daemon-reload <br>
mkdir /raid5/nfs <br>
dnf install nfs-utils nfs4-acl-tools -y <br>
nano /etc/exports

![](/2_mod/6.jpg)

<h1>hq-cli</h1>

mkdir /nfs <br>
nano /etc/exports

![](/2_mod/7.jpg)

mount -a на проверку ошибок

<h1>трансляция портов (hq-rtr, br-rtr)</h1>

nano /etc/sysconfig/nftables

![](/2_mod/8.jpg)

systemctl reload nftables

<h1>moodle (hq-srv)</h1>

dnf install httpd mariadb-server -y <br>
mysql -u root <br>
create database moodledb; <br>
create user moodle; <br>
grant all privileges on moodledb.* to moodle identified by ‘P@ssw0rd’; <br>
cd /tmp <br>
curl -o moodle.zip repos.local/moodle.zip <br>
dnf install zip -y <br>
unzip moodle.zip <br>
cp -r /tmp/moodle/* /var/www/html/ <br>
dnf install php -y <br>
systemctl enable --now httpd <br>
mkdir /var/www/moodledata <br>
chmod 777 /var/www/moodledata <br>
dnf install php-mysqlnd -y <br>
nano /etc/php.ini <br>
через ctrl w ищешь max_input_vars и меняешь на 5000  

![](/2_mod/9.jpg)

systemctl restart httpd <br>
на hq-cli подключаешься к moodle <br>
если необходимо докачиваешь пакеты, названия будут указаны на этапе развертывания на hq-cli

<h1>nginx (hq-rtr)</h1>

dnf install nginx -y  <br>
nano /etc/nginx/nginx.conff

![](/2_mod/10.jpg)

systemctl enable –now nginx

<h1>yandex (hq-cli)</h1>

перейти в загрузки <br>
sudo dnf install ./yandex-browser-stable-24.12.4

<h1>ansible (br-srv)</h1>

dnf install ansible -y <br>
ssh-keygen <br>
ssh-copy-id user@br-rtr (и другие пользователи по заданию) <br>
nano /etc/ansible/hosts

![](/2_mod/11.jpg)

ansible all -m ping <br>
если ping пришел не ко всем, то проверить ping до нужной машины командой <br>
ansible [hostname] -m ping (например, ansible hq-cli -m ping)

<h1>freeIPA(br-srv)</h1>

nano /etc/hosts

![](/2_mod/12.jpg)

dnf install ipa-server pki-ca pki-kra pki-acme <br>
ipa-server-install <br>
ввести пароли и на вопросе сохранения конфигурации ввести yes
