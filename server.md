# Setup Ubuntu server 16.04
First if we install clear sever we need to login as root and update all
## 0. update and upgrade
```
apt-get update && apt-get upgrade
```
## 1. installation VSFTPD
install:
```
apt-get install vsftpd ftp
```
edit `/etc/vsftpd.conf`
```
vim /etc/vsftpd.conf
```
My config is:
```
listen=NO
listen_ipv6=YES
anonymous_enable=NO
local_enable=YES
write_enable=YES
#local_umask=022
#anon_upload_enable=YES
#anon_mkdir_write_enable=YES
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
# If you want, you can arrange for uploaded anonymous files to be owned by
# a different user. Note! Using "root" for uploaded files is not
# recommended!
#chown_uploads=YES
#chown_username=whoever
#xferlog_file=/var/log/vsftpd.log #default
#xferlog_std_format=YES
idle_session_timeout=600
data_connection_timeout=600
# It is recommended that you define on your system a unique user which the
# ftp server can use as a totally isolated and unprivileged user.
# nopriv_user=ftpsecure #default
#async_abor_enable=YES
#ascii_upload_enable=YES
#ascii_download_enable=YES
ftpd_banner=Welcome from NodeJS server.
#deny_email_enable=YES
#banned_email_file=/etc/vsftpd.banned_emails
# You may restrict local users to their home directories.  See the FAQ for
# the possible risks in this before using chroot_local_user or
# chroot_list_enable below.
#chroot_local_user=YES
#chroot_local_user=YES
#chroot_list_enable=YES
#chroot_list_file=/etc/vsftpd.chroot_list
#ls_recurse_enable=YES
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
utf8_filesystem=YES
```
restart to accept new settings:
```
systemctl restart vsftpd
```
create shell script for our ftp connection to avoid firewall. open /etc/shells
```
vim /etc/shells
```
and add to last line
```
/bin/ftpshell
```
create ftp user `ftpadmin`. dont forget to change name in the end
```
groupadd ftp-users
useradd -m -g ftp-users -s /bin/ftpshell ftpadmin
passwd ftpadmin
```
to look ip address:
```
ifconfig
```
and try to connect to port `21`
## NodeJS & yarn
#### Node:
```
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```
#### yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt update
sudo apt install yarn
```
## Nginx
```
apt-get install nginx
```
to restart `nginx` (if you needed) use command `sudo systemctl restart nginx`
### letsencrypt
Make two main files:
1. `/etc/nginx/snippets/letsencrypt.conf`
```
location ^~ /.well-known/acme-challenge/ {
	default_type "text/plain";
	root /var/www/letsencrypt;
}
```
2. `/etc/nginx/snippets/ssl.conf`
```
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets off;

ssl_protocols TLSv1.2;
ssl_ciphers EECDH+AESGCM:EECDH+AES;
ssl_ecdh_curve secp384r1;
ssl_prefer_server_ciphers on;

ssl_stapling on;
ssl_stapling_verify on;

add_header Strict-Transport-Security "max-age=15768000; includeSubdomains; preload";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
```
I'll use directory `/var/www/letsencrypt`, you free to use what you want
```
mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
```
### install certbot
```
apt-get install software-properties-common
add-apt-repository ppa:certbot/certbot
apt-get update
apt-get install certbot
```
### connect sites to the server:
Remove default settings
```
rm /etc/nginx/sites-enabled/default
```
For each domain make such config. Here is for `/etc/nginx/sites-available/mydomain.conf`
```
server {
	listen 80;
	listen [::]:80;
	server_name mydomain.com www.mydomain.com;

	include /etc/nginx/snippets/letsencrypt.conf;

	location / {
		return 301 https://www.mydomain.com$request_uri;
	}
}

server {
	server_name www.mydomain.com;
	listen 443 ssl http2;
	listen [::]:443 ssl http2;

	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	include /etc/nginx/snippets/ssl.conf;

	root /var/www/mydomain.com/public;
	index /../bin/www;
	location / {
		proxy_pass http://111.111.111.111:3000;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
	}
}

server {
	listen 443 ssl http2;
	listen [::]:443 ssl http2;
	server_name mydomain.com;

	ssl_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/www.mydomain.com/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/www.mydomain.com/fullchain.pem;
	include /etc/nginx/snippets/ssl.conf;

	location / {
		return 301 https://www.mydomain.com$request_uri;
	}
}
```
Dont forget to change port and ip address. Lines with ` /etc/letsencrypt/live/` we can connect to single domain (use the same domain each time) cuz we grab single cert for all sites.

And turn them on. There is on symlink for mydomain.com:
```
ln -s /etc/nginx/sites-available/mydomain.conf /etc/nginx/sites-enabled/mydomain.conf
```
After all reload the server:
```
sudo systemctl reload nginx
```
gather certs for domains:
```
certbot certonly --webroot --agree-tos --no-eff-email --email YOUR@EMAIL.COM -w /var/www/letsencrypt -d www.domain.com -d domain.com -d www.domain2.com -d domain2.com -d www.domain3.com -d domain3.com
```
each time if you get more domains add it to list like above. if all done right site will up with `https`
## Install mongo
Here is perfect guide on [official website ->](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/)
## Firewall
Link to [fail2ban](https://github.com/fail2ban/fail2ban) repo
## pm2
install
```
npm i -g pm2
```
and use
```
pm2 start "/usr/local/bin/npm" --name "mydomain" -- start -e /var/logs/pm2-logs/mydomain/err.log -o /var/www/logs/pm2-logs/mydomain/out.log
```
if the path of a npm is wrong - use command `which npm`
