# Implementasi-Docker-Multicontainer-

>>>Step 1 install Docker CE

$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

$ sudo apt-key fingerprint 0EBFCD88

$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

$ sudo apt update
$ sudo apt install docker-ce

==> menjalankan docker tanpa hak ases SUDO

sudo usermod -aG docker ${USER}
su - ${USER}
id -nG
==> Pengenalan Perintah-perintah Docker
docker ps
docker ps -a
docker images
docker pull [name image]
docker container ps
docker container ls
docker container ps -all
docker rm [id container / nama container]
docker image rm [id image / nama image]
docker container stop [id container / nama container]
docker container start [id container / nama container]
docker container rm [id container / nama container]
docker exec -it [id container] bash
>>>Step 2 Install Docker Compose 
sudo apt install docker-compose -y
docker-compose -v
==>perintah-perintah dasar compose
docker-compose up -d
docker-compose ps
docker-compose down
>>>Step 3 Install Server utama di container sendiri
$ docker run --name apache-proxy --network=my-network --volumes-from proxy-data -p 80:80 -d diouxx/apache-proxy
>>>Step 4 Install Management Docker Portainer
$ docker run -d -p 850:9000 -v /var/run/docker.sock:/var/run/docker.sock portainer/portainer
- setelah selesai install, 
  buka ke browser dan search menggunakan ip:port
  example = 192.168.1.38:850
- pilih Lokal, masukkan username dan password, dan next
>>>Step 4 buat direktori folder baru dan file format [.yml] dan [.conf]
|---wordpress-compose/
| |
| |---docker-compose.yml
| |
| |---nginx/
| |   |---wordpress.conf
| |
| |---db-data/
| |
| |---logs/
| |   |
| |   |---nginx/
| |
| |---wordpress

mkdir -p wordpress-compose
cd wordpress-compose/
touch docker-compose.yml
mkdir -p nginx/
mkdir -p db-data/
mkdir -p logs/nginx/
mkdir -p wordpress/


nano nginx/wordpress.conf
server {
    listen 80;
    server_name [IP Address];

    root /var/www/html;
    index index.php;

    access_log /var/log/nginx/nginx-access.log;
    error_log /var/log/nginx/nginx-error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass wordpress:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}


nano docker-compose.yml
nginx:
    image: nginx:latest
    ports:
        - '90:80'
    volumes:
        - ./nginx:/etc/nginx/conf.d
        - ./logs/nginx:/var/log/nginx
        - ./wordpress:/var/www/html
    links:
        - wordpress
    restart: always
mysql:
    image: mariadb
    ports:
        - '3306:3306'
    volumes:
        - ./db-data:/var/lib/mysql
    environment:
        - MYSQL_ROOT_PASSWORD=aqwe123
    restart: always
wordpress:
    image: wordpress:4.9.6-php7.2-fpm
    ports:
        - '9000:9000'
    volumes:
        - ./wordpress:/var/www/html
    environment:
        - WORDPRESS_DB_NAME=wpdb
        - WORDPRESS_TABLE_PREFIX=wp_
        - WORDPRESS_DB_HOST=mysql
        - WORDPRESS_DB_PASSWORD=aqwe123
    links:
        - mysql
    restart: always
>>>Step 5 pull docker-compose
- masuk ke direktori yang terdapat file docker-compose.yml
- lakukan perintah $ docker-compose up -d
*Lakukan 2kali untuk membuat 2 web cms wordpress dengan port yang berbeda
>>>Step 6 mengatur hosts
$ sudo nano /etc/hosts
tambahkan ip address linux dan nama domain
*nama domain bisa lebih dari satu dengan cara pemisah menggunakan spasi
>>>Step 7 mengatur reverse proxy
- cek container apache2 berjalan
- jika sudah berjalan masuk ke bash
$ docker exec -it [id container] bash
setelah masuk root, lakukan update
@ apt update 
@ apt install nano
@ cd etc/apache2/sites-available
buka file 000-default.conf
@ nano 000-default.conf
-lakukan edit file seperti di bawah ini
<VirtualHost *:80>
        ServerName [domain]
        ServerAlias [domain]

        ProxyPass / http://[ip docker]:[port nginx di docker-compose]/
        ProxyPassReverse / http://[ip docker]:[port nginx di docker-compose]/
</VirtualHost>
-lakukan reload untuk apache2
@ service apache2 reload
setelah itu akan muncul output seperti dibawah ini
[ ok ] Reloading Apache httpd web server: apache2.
-lalu exit dari root
@ exit
>>>Step 8 Remote Wordpress
- Login ke portainer 
- menuju ke Endpoints
- add endpoint
- isi nama remote, Endpoint URL [ip pc]:[port nginx di docker-compose],Public IP
 example URL: 192.168.100.33:90
 example Public IP: 192.168.100
- klik add point
>>>Step 9
- buka di browser sesuai dengan domain yang telah diatur
- jika dari laptop lain ingin membuka harus mengatu hosts terlebih dahulu
- tambah ip addres pc yang membuat docker dan nama domain yang sama
example : 192.168.100.33  dockerpkl.id
- save, dan search menggunakan domain
* harus dalam satu jaringan 
