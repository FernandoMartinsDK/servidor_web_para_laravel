
# Servidor web para Laravel
Esse tutorial tem como proposta fornecer um 'norte' no momento de configurar um servidor dedicado LINUX para uma aplicação Laravel.

**Foi utilizado como base o sistema Ubuntu 22.04**

Tecnologias utilizadas
1. Composer
2. NGNIX
3. MySql
4. Node Js
5. PHP 8.4
6. Git
7. Redis *(revisar)*

> Durante o tutorial você irá se deparar com termos como `projeto` fazendo referencia ao nome do seu projeto, atente-se aos pontos que contem esses nomes genéricos que são utilizados meramente de forma ilustrativa, troque ou faça ajustes sempre que julgar melhor.

###   UBUNTU
Verifica qual é a versão do sistema

	lsb_release -a
Atualiza os pacotes:

	apt-get update
Instala nova versão dos pacotes

	apt-get upgrade

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

### PHP 8.4
Adiciona o repositorio

	add-apt-repository ppa:ondrej/php -y
	apt-get update
Instala o PHP e as extensões referente

	apt-get install php8.4 php8.4-dev php8.4-fpm php8.4-cli php8.4-xml -y --allow-unauthenticated
Instala outras extensões

  	apt install php8.4-{bcmath,xml,fpm,mysql,zip,intl,ldap,gd,cli,bz2,curl,mbstring,pgsql,opcache,soap,cgi}
Verifica o status

 	systemctl status php8.4-fpm

###### Configura o PHP
	nano /etc/php/8.4/fpm/php.ini

mudar os seguintes parâmetros

    upload_max_filesize = 256M 
    post_max_size = 256M 
    memory_limit = 3072M 
    max_execution_time = 600 
    max_input_vars = 3000 
    max_input_time = 1000

Verifica se tem algum erro no arquivo e reinicia em seguida

	php-fpm8.4 -t 
	service php8.4-fpm restart

### Composer
Instala composer mais atual

	curl -sS https://getcomposer.org/installer -o composer-setup.php
	HASH=`curl -sS https://composer.github.io/installer.sig`
	echo $HASH
	php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
	sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer

Verifica se foi instalado com sucesso com o comando

	composer

### Redis
Para instalar o Redis local basta ir copiando e colando as seguintes linhas
```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis
```
Para o serviço do Redis iniciar sozinho caso o sistema venha a reiniciar
	
	sudo systemctl enable redis-server

Para testar o redis executar o comando 
	
	redis-cli ping
O retorno deve ser : 
```
PONG
```
Instalar extensão do PHP 8.4

	sudo apt-get install php8.4-redis
Reinicia o serviço
	
	service php8.4-fpm restart


Caso o servidor seja reiniciado pode ser necessario inicializar novamente o serviço
```
redis-server --daemonize yes
```


### NODE JS
###### a instalação foi baseado nesse [link](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-20-04)
Vamos instalar o nodejs utilizando um gerenciador de versão
	
	curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
	source ~/.bashrc
Para ver as versões disponiveis utilize o comando

	nvm list-remote

Para instalar a versão que deseja utilize o comando + versão. Como por exemplo

	nvm install v22.14.0

>Para diminuir a necessidade de manutenção procure sempre dar preferencia a última versão LTS disponivel

###### Compilando o código o código
Apesar de não ser obrigatório é sempre recomendado fazer o 'build' do seu front-end, seja utilizando o mix ou o vite.
Vá até a pasta da aplicação 

	cd /var/www/projeto
	
Instale os pacotes

	npm install
	
'Compila' o código utilizando o vite

	npm run build

### Git
Instala o git

	apt-get install git

###### Adicionar a chave ssh do servidor no git
O primeiro passo é criar uma par de chaves na máquina do cliente

	ssh-keygen -t ed25519 -C "meu-outro-projeto"

Após digitar o comando, você deve ver o seguinte resultado:

	Output
	Generating public/private rsa key pair.
	Enter file in which to save the key (/your_home/.ssh/id_rsa):

Basta ir apertando enter para para ir pulando as perguntas, por padrão o par de chaves ficarão gravadas no `.ssh/`. Depois basta dar o comando `cat` no na chave publica, deve ser algo como:

	cat .ssh/id_rsa.pub

"Ativa" a chave publica 

	eval "$(ssh-agent -s)"
	ssh-add ~/.ssh/id_ed25519_outro_projeto

Adicione a chave pública ao arquivo authorized_keys:

	cat ~/.ssh/id_ed25519_outro_projeto.pub >> ~/.ssh/authorized_keys

No github do projeto vá no `Settings` do projeto e procure por `Deploy Keys` na categoria de `Security`, então clique em `Add deploy key`. De um nome para a chave e cole ela

### NGNIX
Instala o servidor web

	apt install nginx

###### Adiciona o projeto laravel
Vá até a pasta onde será adicionado o projeto

	cd /var/www/

Baixe o projeto do gitbub

	git clone git@github.com:usuario/meu-projeto-laravel.git projeto
	cd projeto

Prepa os arquivos da aplicação

	cp .env.example .env
	composer install
	php artisan key:generate
	php artisan storage:link
Editar os atributos do `.env`
	
	APP_NAME=PROJETO
	APP_DEBUG=false
	APP_ENV=production
	APP_URL=https://www.projeto.com.br

	DB_HOST=127.0.0.1
	DB_READ=127.0.0.1
	DB_PORT=3306
	DB_DATABASE=db-projeto
	DB_USERNAME=administrator
	DB_PASSWORD='SenhaSecreta'

### NGNIX
Adiciona as portas ao firewall

	sudo ufw app list
	sudo ufw allow 'Nginx HTTP'
	sudo ufw allow 'Nginx HTTPS'

###### Dá permissão aos diretorios
	sudo chown -R www-data:www-data /var/www/projeto/storage
	sudo chown -R www-data:www-data /var/www/projeto/bootstrap/cache

Caso tenha problemas permissão você pode usar:

	chmod -R 777 /var/www/projeto/

###### Cria o arquivo o novo arquivo de configuração do servidor web
	nano /etc/nginx/sites-available/projeto

Cole o script:

	server {
		listen 80;
  		listen [::]:80;
		server_name IP_DO_SERVIDOR DOMINIO_DO_SITE ;
		root /var/www/projeto/public;
		client_max_body_size 0;

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
			fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;
			fastcgi_index index.php;
			fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
			include fastcgi_params;
		}

		location ~ /\.(?!well-known).* {
			deny all;
		}
	}

Muda o link para o novo script e verifica se tem algum erro

	sudo ln -s /etc/nginx/sites-available/projeto /etc/nginx/sites-enabled/
	sudo nginx -t

###### Para liberar  e criar  sub dominios
Acessar o arquivo

	nano /etc/nginx/nginx.conf

Remova `#` para descomentar o comando `**server_names_hash_bucket_size 64**;`

Caso o novo sub dominio aponte para o **mesmo** projeto basta ir no arquivo de configuração de dominio

	nano /etc/nginx/sites-available/projeto
Adicione o novo sub dominio em na linha `server_name`, ficando:
`server_name DOMINIO_DO_SITE SUB_DOMINIO_DO_SITE`
Caso tenha outro sub dominio para adicionar basta dar um espaço e adicionar

Para finalizar vá no arquivo de host

	nano /etc/hosts	
Adicione o subdominio: `127.0.0.1 admin.example.local`
	
Reinicie o serviço

	sudo systemctl reload nginx

### REINICIA OS SERVIÇOS
	systemctl daemon-reload
	systemctl reload nginx
	service php8.4-fpm restart

### ATIVA O CRON DO SISTEMA
	nano /etc/crontab
Na ultima linha adicione (deixe tudo alinhado dando os espaços necessarios)

    * * * * *  root  cd /var/www/projeto && php artisan schedule:run >> /dev/null 2>&1
verifica se a cron está funcionando

	service cron status
*Caso apresente mensagem de erro é preciso ativar*

	service cron start


### CERTIFICADO  PADRÃO
###### Caso o projeto venha fazer uso de sub dominios é recomendado não utilizar essa opção pois ao tentar atualizar o certificado do subdominio pode vim apresentar erros
*(baseado no link: https://www.linuxtuto.com/how-to-secure-nginx-with-lets-encrypt-on-ubuntu-22-04/)*

	apt install certbot python3-certbot-nginx
	certbot --nginx

### CERTIFICADO  COM SUB-DOMINIO + CLOUDFLARE
Caso o projeto tenha sub-dominio e o DNS esteja **na cloudflare.**
Na Cloudflare faça o seguinte:
Em **DNS** → **Registros**  deixe o registros da seguinte maneira
| Tipo | Nome | Conteúdo | Statusdo proxy | TTL
|--|--|--|--|--|
| A | * | ip_do_servidor | somente DNS | Auto
| A | dominio.com.br | ip_do_servidor | somente DNS | Auto
| CNAME | www | ip_do_servidor | somente DNS | Auto

Agora vamos gerar um toquen de acesso para o servidor pode registrar o certificidados de forma automatica

 1. Acesse Cloudflare Dashboard.
 2. Vá até **Meu Perfil** (canto superior direito).
 3. Clique na aba **Tokens de API**.
 4. Clique em **Criar Token** e selecione **Custom Token**.
 5. Configure as permissões:
 	-   **Zone** → **DNS** → **Edit** (para editar os registros DNS)
	-   **Zone** → **Zone** → **Read** (para ler as informações da zona)
 6. Escolha um escopo que inclua o domínio `dominio.com.br`.
 7. Gere o token e copie-o.
<hr>
Agora entre no servidor via ssh e faça o seguinte

 1. Instalar o Plugin da Cloudflare no Certbot
	Como você usa Cloudflare para gerenciar os DNS, precisará instalar o plugin correspondente no Certbot:
	
		sudo apt update
		sudo apt install python3-certbot-dns-cloudflare
2.  Criar o Arquivo de Configuração para a Cloudflare

		sudo nano /root/.cloudflare.ini

 Adicione o seguinte conteúdo:

	dns_cloudflare_api_token = SEU_TOKEN_AQUI

Salve e depois defina as permissões corretas:

	sudo chmod 600 /root/.cloudflare.ini

3. Emitir o Certificado com Desafio DNS-01
Agora execute o seguinte comando para gerar um novo certificado:

		sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.cloudflare.ini -d dominio.com.br -d "*.dominio.com.br"

	Se tudo correr bem, ele validará os domínios e gerará o certificado corretamente.

4. Configurar a Renovação Automática
Para garantir que o certificado seja renovado automaticamente, adicione o seguinte comando ao `crontab`:

		sudo crontab -e

	Adicione esta linha no final do arquivo:

		0 3 1 * * certbot renew --dns-cloudflare --dns-cloudflare-credentials /root/.cloudflare.ini --quiet
*Agora ele vai atualizar todo dia 1 de cada mês as 3 da manhã*

Por fim basta reinicar o servidor web

	sudo systemctl restart nginx

Agora o wildcard (`*.dominio.com.br`) estará coberto pelo certificado e seus subdomínios vão funcionar corretamente
### ATUALIZAR O CÓDIGO
Para atualizar o código do projeto  vá na pasta do sistema

	cd /var/www/projeto
Para atualizar o código 

	git pull
*Caso o comando git pull não funcione *

	git pull --rebase --autostash

>Existem diversas formas de atualizar o código do servidor, formas mais manuais como FTP a outras automatizadas utilizandos webhook ou GitHub Actions. A forma informada aqui é uma das mais simples e seguras mas existem *melhores* opções.

### Comandos uteis
Para ver todos os serviços que estão sendo executados

	service --status-all
