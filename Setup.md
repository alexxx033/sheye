# Requirements

* PHP >= 8.1
* golang >= 1.9
* laravel 9 
* composer 2



# Packages 

`apt-get install python3-dev python3-pip nginx imagemagick default-jdk wget maven nginx curl unzip xvfb libxi6 libgconf-2-4 python2.7  unzip mariadb-server  git nmap `


**If you use **Ubuntu** based distro**

``` apt get install chromium-chromedriver```

**If you use **Debian** based distro**

``` apt get install chromium-driver```

## PHP 8

You need to install **php8** for laravel to work.

```
apt install php-fpm php-cli php-zip php-mbstring php-xml php-dev php-mysql php-pdo php-curl php-bcmath php-dom php-ctype php-cli
```


If you use **Debian** based distro or **php8** is missing, you can do next

```
sudo apt install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/sury-php.list
curl -fsSL  https://packages.sury.org/php/apt.gpg| sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/sury-keyring.gpg
apt install php8.1-fpm php8.1-cli php8.1-zip php8.1-mbstring php8.1-xml php8.1-dev php8.1-mysql php8.1-pdo php8.1-curl php8.1-bcmath php8.1-dom php8.1-ctype php8.1-cli
```


## Nginx

Configure Nginx to serve the app

```
server {
listen 80;
    server_name _;
    root /var/www/html/public; #change to correct folder
    large_client_header_buffers 4 16k;
    index index.php;

    client_max_body_size 100m;
    charset utf-8;

  location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```



## Golang

You need to install **golang >= 1.9**

### AMD64

```
wget "https://go.dev/dl/go1.19.2.linux-arm64.tar.gz" -4
tar -C /usr/local -xvf "go1.19.2.linux-arm64.tar.gz"
rm "go1.19.2.linux-arm64.tar.gz"
cat >> ~/.bashrc << 'EOF'
export GOPATH=$HOME/go
export PATH=/usr/local/go/bin:$PATH:$GOPATH/bin
EOF

source ~/.bashrc
rm -rf /usr/local/go
```

### ARM64

```
wget "https://go.dev/dl/go1.19.2.linux-amd64.tar.gz" -4
tar -C /usr/local -xvf "go1.19.2.linux-amd64.tar.gz"
rm "go1.19.2.linux-amd64.tar.gz"
cat >> ~/.bashrc << 'EOF'
export GOPATH=$HOME/go
export PATH=/usr/local/go/bin:$PATH:$GOPATH/bin
EOF

source ~/.bashrc
rm -rf /usr/local/go
```





## Composer


```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer
```

# Database

```
mysql -e "UPDATE mysql.user SET Password = PASSWORD('12345') WHERE User = 'root'"
mysql -e "DELETE FROM mysql.user WHERE User='';"
mysql -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');"
mysql -e "DROP DATABASE IF EXISTS test;"
mysql -e "DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';"
mysql -e "create database scanner";
mysql -e "CREATE USER 'scanner'@'localhost' IDENTIFIED BY 'scanner'";
mysql -e "GRANT ALL PRIVILEGES ON scanner . * TO 'scanner'@'localhost'";
mysql -e "FLUSH PRIVILEGES";
```


# Setup

Download the **main** branch or **release** version and extract sources to folder, F.E **/var/www/html**

Navigate **/var/www/html**

Make some folders and files writable
```
php artisan storage:link
chmod 777 -R storage/
chmod 777 *.log
```

Run composer, migrations,seeders and add **admin user** from **/var/www/html**

```
composer install
php artisan migrate
php artisan db:seed --class=ScannerSeeder
php artisan add:user admin admin
```

# Scanners


All scanners should be located in **scanners** folder (**/var/www/html/scanners**). You can find example compilation for **arm64**, just change to **amd64** for x64 architecture.

## **amass**

Install **amass** to **scanners/amass_linux** folder.


### ARM64

```
wget https://github.com/OWASP/Amass/releases/download/v3.20.0/amass_linux_arm64.zip
unzip amass_linux_arm64.zip
mv amass_linux_arm64 amass_linux
rm -f amass_linux_arm64.zip
```


### AMD64

```
wget https://github.com/OWASP/Amass/releases/download/v3.20.0/amass_linux_amd64.zip
unzip amass_linux_arm64.zip
mv amass_linux_arm64 amass_linux
rm -f amass_linux_arm64.zip
```



## assetfinder


### ARM64 

```
git clone https://github.com/tomnomnom/assetfinder asset
cd asset
go mod init go.mod
go mod tidy
GOOS=linux GOARCH=arm64 go build -o assetfinder
mv assetfinder /var/www/html/scanners
```

### AMD64

```
git clone https://github.com/tomnomnom/assetfinder asset
cd asset
go mod init go.mod
go mod tidy
GOOS=linux GOARCH=amd64 go build -o assetfinder
mv assetfinder /var/www/html/scanners
```



## subfinder


### ARM64 

```
wget https://github.com/projectdiscovery/subfinder/releases/download/v2.5.4/subfinder_2.5.4_linux_arm64.zip
unzip subfinder_2.5.4_linux_arm64.zip
mv subfinder/var/www/html/scanners/
rm -f subfinder_2.5.4_linux_arm64.zip
```


### AMD64 

```
wget https://github.com/projectdiscovery/subfinder/releases/download/v2.5.4/subfinder_2.5.4_linux_amd64.zip
unzip subfinder_2.5.4_linux_amd64.zip
mv subfinder/var/www/html/scanners/
rm -f subfinder_2.5.4_linux_amd64.zip
```



## httpx

```
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
mv /root/go/bin/httpx /var/www/html/scanners/
```

## nuclei
```
go install -v github.com/projectdiscovery/nuclei/v2/cmd/nuclei@latest
mv /root/go/bin/nuclei /var/www/html/scanners/
```

## dnsx
```
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
mv /root/go/bin/dnsx /var/www/html/scanners/
```

## gau
```
go install github.com/lc/gau/v2/cmd/gau@latest
mv /root/go/bin/gau /var/www/html/scanners/
```

## screenshot

The screenshot is a custom version of https://github.com/zzzteph/screenshot. Navigate **/var/www/html/scanners/screenshot**


```
mvn package
```

## dirsearch

```
git clone https://github.com/maurosoria/dirsearch.git --depth 1
cd dirsearch
pip3 install -r requirements.txt
cd ../
chmod 777 -R dirsearch
```


# Workers

Workers are located at **workers.sh** file. You must launch that file in the background or with **screen**.

```
cd /var/www/html
screen -S scanner
sh worker.sh
```