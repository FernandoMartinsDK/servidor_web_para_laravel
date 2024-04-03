# Servidor web para Laravel
Esse tutorial tem como proposta fornecer um 'norte' no momento de configurar um servidor dedicado LINUX para uma aplicação Laravel.
**Esse tutorial utilizou como base o sistema Ubuntu 22.04**

Tecnologias utilizadas
1. Composer
2. NGNIX
3. MySql
4. Node (*não adicionado*)
5. PHP 8.3
6. Git


> Durante o tutorial você irá se deparar com termos como `projeto` fazendo referencia ao nome do seu projeto, atente-se aos pontos que contem esses nomes genéricos que são utilizados meramente de forma ilustrativa, troque ou faça ajustes sempre que julgar melhor.

###   UBUNTU
Verifica qual é a versão do sistema

	lsb_release -a
Atualiza os pacotes:

	apt get update
Instala nova versão dos pacotes

	apt get upgrade

### Definindo fuso horario
	timedatectl set-timezone America/Sao_Paulo

### Instalar curl para acessar sites externos
	sudo apt install curl


### Firewall
Ativa o firewal e adiciona as portas

    ufw allow 3306/tcp
    sudo ufw allow 20/tcp
    sudo ufw allow 21/tcp
    sudo ufw allow 22/tcp
    sudo ufw allow 40000:50000/tcp
    sudo ufw allow 990/tcp
    sudo ufw enable

### MYSQL
Instala o MySql Server

	apt install mysql-server
libera gravação e acesso de aplicativos acendo o arquivo de configuração em:

	nano /etc/mysql/mysql.conf.d/mysqld.cnf 
Na linha que esta escrito " bind-address = 127.0.0.1 " mude para " bind-address = 0.0.0.0 ". Salve a modificação e sai do arquivo

Para se criar um usuário que possa acessar de qualquer host e fazer modificações no banco
mysql

    CREATE USER 'administrator'@'%' IDENTIFIED BY 'SenhaSecreta';
    GRANT ALL PRIVILEGES ON *.* TO 'administrator'@'%' WITH GRANT OPTION;
    FLUSH PRIVILEGES;
    exit
Atualiza o serviço do MySql

	/etc/init.d/mysql restart
### PHP 8.3
Adiciona o repositorio

	add-apt-repository ppa:ondrej/php -y
	apt-get update
Instala o PHP e as extensões referente

	apt-get install php8.3 php8.3-dev php8.3-fpm php8.3-cli php8.3-xml -y --allow-unauthenticated
 	apt install php8.3-{bcmath,xml,fpm,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi}
	systemctl status php8.3-fpm

###### Configura o PHP
	nano /etc/php/8.3/fpm/php.ini

mudar os seguintes parâmetros

    upload_max_filesize = 32M 
    post_max_size = 48M 
    memory_limit = 256M 
    max_execution_time = 600 
    max_input_vars = 3000 
    max_input_time = 1000

Verifica se tem algum erro no arquivo e reinicia em seguida

	php-fpm8.3 -t 
	service php8.3-fpm restart

### Composer
Instala composer mais atual

	curl -sS https://getcomposer.org/installer -o composer-setup.php
	HASH=`curl -sS https://composer.github.io/installer.sig`
	echo $HASH
	php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
	sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

Verifica se foi instalado com sucesso
	composer

### Git
Instala o git

	apt-get install git

###### Adicionar a chave ssh do servidor no git
*(Baseado no link https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04-pt)*
O primeiro passo é criar uma par de chaves na máquina do cliente

	ssh-keygen

Por padrão, as versões mais recentes do ssh-keygen criarão um par de chaves RSA de 3072 bits, que é seguro o suficiente para a maioria dos casos de uso (você pode usar o sinalizador `-b 4096` para criar uma chave maior de 4096 bits).
Após digitar o comando, você deve ver o seguinte resultado:

	Output
	Generating public/private rsa key pair.
	Enter file in which to save the key (/your_home/.ssh/id_rsa):

Basta ir apertando enter para para ir pulando as perguntas, por padrão o par de chaves ficarão gravadas no `.ssh/`. Depois basta dar o comando `cat` no na chave publica, deve ser algo como:

	cat .ssh/id_rsa.pub

No github do projeto vá no `Settings` do projeto e procure por `Deploy Keys` na categoria de `Security`, então clique em `Add deploy key`. De um nome para a chave e cole ela

###### Adiciona o projeto laravel
Vá até a pasta onde será adicionado o projeto

	cd /var/www/

Baixe o projeto do gitbub

	git clone git@github.com:usuario/meu-projeto-laravel.git projeto
	cd ponto

Prepa os arquivos da aplicação

	cp .env.example .env
	composer install
	php artisan key:generate
	php artisan storage:link

### NGNIX
Instala o servidor web
	apt install nginx

Adiciona as portas ao firewall
	sudo ufw app list
	sudo ufw allow 'Nginx HTTP'
	sudo ufw allow 'Nginx HTTPS'

###### Dá permissão aos diretorios
	sudo chown -R www-data.www-data /var/www/projeto/storage
	sudo chown -R www-data.www-data /var/www/projeto/bootstrap/cache

Caso tenha problemas permissão você pode usar:

	chmod -R 777 /var/www/projeto/

###### Cria o arquivo o novo arquivo de configuração do servidor web
	nano /etc/nginx/sites-available/projeto

Cole o script:

	server {
		listen 80;
		server_name IP_DO_SERVIDOR DOMINIO_DO_SITE ;
		root /var/www/projeto/public;

		add_header X-Frame-Options "SAMEORIGIN";
		add_header X-XSS-Protection "1; mode=block";
		add_header X-Content-Type-Options "nosniff";

		index index.html index.htm index.php;

		charset utf-8;

		location / {
			try_files $uri $uri/ /index.php?$query_string;
		}

		location = /favicon.ico { access_log off; log_not_found off; }
		location = /robots.txt  { access_log off; log_not_found off; }

		error_page 404 /index.php;

		location ~ \.php$ {
			fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
			include fastcgi_params;
		}

		location ~ /\.(?!well-known).* {
			deny all;
		}
	}

Muda o link para o novo script e reinicia o serviço

	sudo ln -s /etc/nginx/sites-available/projeto /etc/nginx/sites-enabled/
	sudo nginx -t
	sudo systemctl reload nginx

### CERTIFICADO 
###### Aqui precisa de uma revisão melhor, pois este certificado apresenta erro ao se atualizar para subdominios
*(baseado no link: https://www.linuxtuto.com/how-to-secure-nginx-with-lets-encrypt-on-ubuntu-22-04/)*

	apt install certbot python3-certbot-nginx
	certbot --nginx

### REINICIA OS SERVIÇOS
	systemctl daemon-reload
	systemctl reload nginx
	service php8.3-fpm restart

### ATIVA O CRON DO SISTEMA
	nano /etc/crontab
Na ultima linha adicione (deixe tudo alinhado dando os espaços necessarios)
    * * * * *  root  cd /var/www/projeto && php artisan schedule:run >> /dev/null 2>&1
verifica se a cron está funcionando

	service cron status
*Caso apresente mensagem de erro é preciso ativar*

	service cron start

### ATUALIZAR O CÓDIGO
Para atualizar o código do projeto  vá na pasta do sistema

	cd /var/www/ponto
Para atualizar o código 

	git pull
*Caso o comando git pull não funcione *

	git pull --rebase --autostash

>Existe diversas formas de atualizar o código do servidor, formas mais manuais como FTP a outras automatizadas utilizandos webhook ou GitHub Actions. A forma informada aqui é uma das mais simples e seguras mas existem *melhores* opções.

