Instalando o docker

 apt-get install curl  apt-transport-https
 
##Acessando o repositório
 ```bash
nano  /etc/apt/sources.list
```

## Alterando o repositório
```bash
deb http://ftp.br.debian.org/debian/ stretch main contrib non-free
deb http://security.debian.org/debian-security stretch/updates main
deb http://ftp.br.debian.org/debian/ stretch-updates main
```

## Configurando Docker
> repositório do docker
```bash
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

> Chave pública do docker
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg |  gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```


##Atualizando repositórios
```bash
apt-get update
```

## Instalação de programas necessários.
```bash
apt-get install ca-certificates  gnupg  lsb-release docker-ce docker-ce-cli containerd.io docker-compose -y
```

docker pull mysql:5.7

docker images

docker run -e MYSQL_ROOT_PASSWORD=root --name mysql-docker -d -p 3306:3306 mysql:5.7

create database linux_dio;
use linux_dio;

CREATE TABLE dados (
    AlunoID int,
    Nome varchar(50),
    Sobrenome varchar(50),
    Endereco varchar(150),
    Cidade varchar(50),
    Host varchar(50)
);

mkdir /var/lib/docker/volumes/app
mkdir /var/lib/docker/volumes/app/_data
nano /var/lib/docker/volumes/app/_data/index.php

```bash
<html>

<head>
<title>Exemplo PHP</title>
</head>
<body>

<?php
ini_set("display_errors", 1);
header('Content-Type: text/html; charset=iso-8859-1');



echo 'Versao Atual do PHP: ' . phpversion() . '<br>';

$servername = "172.17.0.1";
$username = "root";
$password = "root";
$database = "linux_dio";

// Criar conexão


$link = new mysqli($servername, $username, $password, $database);

/* check connection */
if (mysqli_connect_errno()) {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
}

$valor_rand1 =  rand(1, 999);
$valor_rand2 = strtoupper(substr(bin2hex(random_bytes(4)), 1));
$host_name = gethostname();


$query = "INSERT INTO dados (AlunoID, Nome, Sobrenome, Endereco, Cidade, Host) VALUES ('$valor_rand1' , '$valor_rand2', '$valor_rand2', '$valor_rand2', '$valor_rand2','$host_name')";


if ($link->query($query) === TRUE) {
  echo "New record created successfully";
} else {
  echo "Error: " . $link->error;
}

?>
</body>
</html>

```


docker run --name web-server -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7

docker rm --force web-server

docker swarm init 

docker swarm join --token SWMTKN-1-3kl2s6qc3sq3q353uz27xbh2v7cwp0qxrmgmx8kf5bc17k8w75-79o1bti0mxu2utatm28but7qk 192.168.1.105:2377

docker node ls


docker service create --name web-server --replicas 9 -dt -p 80:80 --mount type=volume,src=app,dst=/app/ webdevops/php-apache:alpine-php7

docker service ps web-server

apt-get install nfs-server // instalar no servidor

apt-get install nfs-common  // instalar no cliente

nano /etc/exports
```bash
/var/lib/docker/volumes/app/_data *(rw,sync,subtree_check)
```

exportar a pasta
exportfs -ar 

///visualizar pasta compartilhada
showmount -e


mount -o v3 192.168.1.105:/var/lib/docker/volumes/app/_data /var/lib/docker/volumes/app/_data


# criando um proxy utilizando o NGINX

mkdir /proxy
nano /proxy/nginx.conf

```bash
http {
   
    upstream all {
        server 192.168.1.105:80;
        server 192.168.1.106:80;
        server 192.168.1.107:80;
    }

    server {
         listen 4500;
         location / {
              proxy_pass http://all/;
         }
    }

}


events { }
```

nano /proxy/dockerfile

```bash
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

docker build -t proxy-app .


docker container run --name my-proxy-app -dti -p 4500:4500 proxy-app
