
(installation)=
# Installation


## Déployer rapidement sur sa propre infrastructure

### Puis je tester rapidement cette outil sur ma propre machine?

Nous fournissons des [releases packagés](https://github.com/correctexam/corrigeExamBack/releases) pour fonctionner sur les trois systèmes d'exploitation (windows, macos, linux) pour les architecture AMD64 muni d'une base de données intégré. 

Sous linux et macos, vous pouvez juste télécharger le binaire pour votre os, rendre se binaire exécutable, lancer l'application et aller sur votre navigateur à l'adresse http://localhost:8080 (utilisateur par défaut user/user ou admin/admin).

Sous windows, il sera nécessaire de télécharger l'exécutable mais aussi les deux fichiers *mydb...* correspondant à la base de données. Placer ces trois fichiers dans le même répertoire et lancer l'exécutable.  Aller sur votre navigateur à l'adresse http://localhost:8080 (utilisateur par défaut user/user ou admin/admin).

:::{note}
Les données du projet peuvent ensuite être exportées puis importées sur la plateforme en ligne, entres autres si vous souhaitez tester l'envoi de mails aux étudiants pour qu'ils consultent leur copies.
:::

### Puis-je utiliser le service d'authentification de l'Université ou de mon école

La version actuelle supporte une authentification derrière un CAS. Quarkus est facilement configurable pour utiliser un autre système d'authentification unifiée type keycloak.

### Puis-je tester une instance de cette application sur ma propre infrastructure (je veux garder mes données privées) ?

Oui, veuillez consulter la documentation ci dessous. Nous fournissons des scripts pour déployer cette application sur tout type d'infrastructure, d'un serveur puissant avec K8S à un raspberry 4. 

Vous pouvez aussi facilement la tester localement. Vous avez juste besoin de docker.

```bash
git clone -b develop https://github.com/correctexam/corrigeExamBack
cd corrigeExamBack/src/main/docker
docker-compose -f app.yml build --no-cache  back front
docker-compose -f app.yml up
```

l'application est alors disponible sur [http://localhost:9000](http://localhost:9000)
la partie phpmyadmin est disponible sur [http://localhost:91](http://localhost:91)
la partie faux serveur de messagerie est disponible sur [http://localhost:9000/maildev/](http://localhost:9000/maildev/)


Pour plus d'informations :

1. Si vous souhaitez vous connecter à un partage de type serveur de messagerie réel, vous pouvez avoir un aperçu des propriétés de quarkus sous forme de commentaire dans le fichier *app.yaml*.

2. Si vous voulez changer les ports
  
- pour phpadmin,
  - pour le port hôte, vous devez le modifier dans le fichier *app.yml  
  - pour le port interne, vous devez le changer à la fois dans le fichier *app.yml* et dans le fichier *myadmin.conf* (fichier du conteneur nginx)
- pour l'application
  - pour le port hôte, vous devez le modifier dans le fichier app.yml.
  - pour le port interne, vous devez le changer dans le fichier app.yml et dans le fichier exam.conf (fichier du conteneur nginx) pour le front et mettre à jour les propriétés de quarkus si vous voulez changer le port interne du back (pas de raison réelle).

1. Si vous voulez mettre en place un reverse proxy en fonction d'un nom de domaine, tout se passera dans les fichiers *exam.conf* et *myadmin.conf* mais il faut aussi mettre à jour l'url externe dans l'application, c'est une propriété quarkus dans le fichier app.yml.



## Construire le projet pour l'archirecture AMD64

### Construire le projet

####  Construire le  Backend

Si vous souhaitez le construire manuellement, vous pouvez simplement exécuter la commande suivante :

```bash
git clone https://github.com/correctexam/corrigeExamBack
cd corrigeExamBack
mvn -B package --file pom.xml -Pnative
docker build -f src/main/docker/Dockerfile.native -t barais/correctexam-back:manifest-amd64 --build-arg ARCH=amd64/  .
```

Le backend est également construit automatiquement en utilisant l'action github. Vous pouvez accéder à l'image du backend dans [docker hub](https://hub.docker.com/repository/docker/barais/grade-scope-istic)

**OR** 

```bash
#if you install the quarkus cli
git clone https://github.com/correctexam/corrigeExamBack
cd corrigeExamBack
quarkus build --native --no-tests -Dquarkus.native.container-build=true  
docker build -f src/main/docker/Dockerfile.native -t barais/correctexam-back:manifest-amd64 --build-arg ARCH=amd64/  .
```

####  Construire la partie Front

**Sans docker**
Il suffit de cloner le projet

:::{attention}
⚠️ mettre à jour webpack/environment.js avec votre nom de domaine.
:::



```bash
# require nodejs v16 you can install it using nvm (https://github.com/nvm-sh/nvm)
git clone https://github.com/correctexam/corrigeExamFront
cd corrigeExamFront
# update webpack/environment.js with your domain names
npm install 
npm run webapp:build:prod
## You can be inspired by webapp:build:prodgithubpage task if you want to manage a prfix for your webapp. 
## if you want to deploy on github page, you can be inspired by the provided github action
```

**Avec docker**

Pour construire le front, nous fournissons un simple fichier docker. 

:::{attention}
⚠️ mettre à jour webpack/environment.js avec votre nom de domaine.
:::


```bash
git clone https://github.com/correctexam/corrigeExamFront
cd corrigeExamFront
# update webpack/environment.js with your domain name
sudo docker build -f src/main/docker/Dockerfile -t barais/correctexam-front:manifest-amd64 --build-arg ARCH=amd64/ .
# OR 
sudo docker buildx build  -f src/main/docker/Dockerfile --push --platform linux/arm64,linux/amd64  --tag barais/correctexam-front .

```

Vous obtiendrez un nginx avec seulement js, html et js. Vous devez monter la configuration si vous voulez gérer le proxy vers les routes du backend. Je préférerais utiliser un nginx bunkerisé pour le routage.

### Déployez tout sur votre propre infrastructure



```yaml
version: '2'
services:
  correctexam-back:
    image: barais/correctexam-back:manifest-amd64
    volumes:
# Path for 
      - /tmp/files:/tmp/files:rw 
    restart: always
    ports:
      - 8080:8080
# All quarkus configuration parameters (knobs) could be override through command line. You could also use different options to update the configuration parameters.  https://quarkus.io/guides/config-reference
    command: ./application -Dquarkus.http.host=0.0.0.0 -Dquarkus.datasource.username=root -Dquarkus.datasource.password='' -Dquarkus.datasource.jdbc.url=jdbc:mysql://correctexam-mysql:3306/correctexam?useUnicode=true&characterEncoding=utf8&useSSL=false -Dquarkus.http.cors=true -Dquarkus.http.cors.origins=https://correctexam.github.io -Dquarkus.http.cors.methods=GET,PUT,POST,DELETE,PATCH,OPTIONS -Dquarkus.http.cors.headers=accept,origin,authorization,content-type,x-requested-with -Dquarkus.http.cors.exposed-headers=Content-Disposition -Dquarkus.http.cors.access-control-max-age=24H -Dquarkus.mailer.from=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.host=partage.univ-rennes1.fr -Dquarkus.mailer.port=587 -Dquarkus.mailer.ssl=false -Dquarkus.mailer.username=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.password=TOCHANGE -Djhipster.mail.base-url=https://correctexam.github.io/corrigeExamFront
  correctexam-mysql:
    image: mysql:8.0.20
    volumes:
      - ./../resources/db/migration/:/docker-entrypoint-initdb.d
    environment:
      - MYSQL_USER=root
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
      - MYSQL_DATABASE=correctexam
    command: mysqld --lower_case_table_names=1 --skip-ssl --character_set_server=utf8mb4 --explicit_defaults_for_timestamp
#    ports:
#      - 3308:3306
  front:
    image: barais/correctexam-front:manifest-amd64
#    ports:
#      - 90:80
    volumes:
      -  ./exampleconf/exam.conf:/etc/nginx/conf.d/exam.conf
      -  ./exampleconf/nginx.conf:/etc/nginx/nginx.conf:ro
```


**exam.conf** et **nginx.conf** pourraient ressembler à quelque chose comme ça (vous devez mettre à jour le nom du serveur)

**exam.conf**


```nginx
server {
    listen       80;
    listen  [::]:80;
    # server name to change based on your own domain name for your front
    server_name  correctexam.barais.fr;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location /api {
        proxy_pass http://correctexam-back:8080/api;
        proxy_set_header Host $http_host;

    }

    location /api {
        include proxy_params;
    	proxy_pass http://correctexam-back:8080/api;
    }

    location /management {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/management;
    }

    location /swagger-ui {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/swagger-ui;
    }

    location /v3/api-docs {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/v3/api-docs;
    }

    location /auth {
	include proxy_params;
        proxy_pass http://correctexam-back:8080/auth;

    }

    location /health {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/health;
    }

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html?$args;

    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

**nginx.conf**

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 1000s;
	types_hash_max_size 2048;


    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;
	client_max_body_size 900M;
	client_body_buffer_size     900M;

    include /etc/nginx/conf.d/*.conf;
}
```

### Où déployer la base de données et le backend sur votre propre infrastructure et le frontend sur un CDN

Si vous souhaitez déployer la base de données et l'infrastructure backend sur votre propre infrastructure et déployer le frontend sur le CDN. Vous devez gérer correctement les autorisations CORS au sein de votre CDN et de votre backend. Si vous utilisez la page publique de github, Pages autorise CORS (l'en-tête access-control-allow-origin est positionné à *). Pour le backend, vous pouvez utiliser les propriétés quarkus *-Dquarkus.http.cors=true -Dquarkus.http.cors.origins=https://correctexam.github.io -Dquarkus.http.cors.methods=GET,PUT,POST,DELETE,PATCH,OPTIONS* pour gérer vos cors. Veuillez mettre à jour le descripteur docker-compose en conséquence. 

Pour démarrer votre backend et votre frontend, je vais vous proposer le squelette d'un docker-compose. Veuillez le mettre à jour en fonction de vos besoins. 
En plus de cela, vous pouvez configurer nginx bunkerisé pour configurer automatiquement votre sécurité et votre certificat let's encrypt. Ce docker-compose configure automatiquement la base de données et le backend. Vous devez mettre à jour le fichier sql ./../resources/db/migration/ qui sera exécuté au démarrage de la base de données. Il remplit la base de données avec les données initiales.


```yaml
version: '2'
services:
  correctexam-back
    image: barais/correctexam-back:manifest-amd64
    volumes:
# Path for 
      - /tmp/files:/tmp/files:rw 
    restart: always
    ports:
      - 8080:8080
# All quarkus configuration parameters (knobs) could be override through command line. You could also use different options to update the configuration parameters.  https://quarkus.io/guides/config-reference
    command: ./application -Dquarkus.http.host=0.0.0.0 -Dquarkus.datasource.username=root -Dquarkus.datasource.password='' -Dquarkus.datasource.jdbc.url=jdbc:mysql://correctexam-mysql:3306/correctexam?useUnicode=true&characterEncoding=utf8&useSSL=false -Dquarkus.http.cors=true -Dquarkus.http.cors.origins=https://correctexam.github.io -Dquarkus.http.cors.methods=GET,PUT,POST,DELETE,PATCH,OPTIONS -Dquarkus.http.cors.headers=accept,origin,authorization,content-type,x-requested-with -Dquarkus.http.cors.exposed-headers=Content-Disposition -Dquarkus.http.cors.access-control-max-age=24H -Dquarkus.mailer.from=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.host=partage.univ-rennes1.fr -Dquarkus.mailer.port=587 -Dquarkus.mailer.ssl=false -Dquarkus.mailer.username=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.password=TOCHANGE -Djhipster.mail.base-url=https://correctexam.github.io/corrigeExamFront
  correctexam-mysql:
    image: mysql:8.0.20
    volumes:
      - ./../resources/db/migration/:/docker-entrypoint-initdb.d
    environment:
      - MYSQL_USER=root
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
      - MYSQL_DATABASE=correctexam
    command: mysqld --lower_case_table_names=1 --skip-ssl --character_set_server=utf8mb4 --explicit_defaults_for_timestamp
    ports:
      - 3308:3306
```

:::{attention}
Avant de construire le frontend pour votre CDN, n'oubliez pas de mettre à jour **webpack/environment.js** avec vos noms de domaine.
:::


##  Construire et déployer sur raspberry PI (arm64)


###  Install support of cross compile on your machine


```bash 
sudo apt-get install qemu binfmt-support qemu-user-static
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes # This step will execute the registering scripts
docker run --rm -t arm64v8/ubuntu uname -m 
```

### Construire le projet

#### Construire le backend

Créer un fichier docker pour quarkus afin d'effectuer une compilation croisée sur votre machine pour arm64

```dockerfile
FROM ghcr.io/graalvm/graalvm-ce:ol8-java11@sha256:dc9effae9a92d50e0a173f1cb8113409a4b6d7fb0c44fcf2195f0e03d6161bc5 AS build
RUN gu install native-image
WORKDIR /project
VOLUME ["/project"]
ENTRYPOINT ["native-image"] 
```

Construire l'image

```bash 
docker build -f src/main/docker/Dockerfile.build.aarch64 -t barais/quarkus-build-aarch64 .
```

Construisez ensuite votre exécutable

```bash
./mvnw clean package -Pnative -Pprod -Dquarkus.package.type=native -DskipTests=true  -Dquarkus.native.container-build=true -Dquarkus.native.builder-image=barais/quarkus-build-aarch64:latest
```

**OU** le construire à l'aide de quarkus cli

```bash
quarkus build --native --no-tests -Dquarkus.native.container-build=true -Dquarkus.native.builder-image=barais/quarkus-build-aarch64
```

Lorsque la compilation est terminée (ce qui peut prendre beaucoup de temps ;), vous pouvez construire les images finales avec le binaire.


**Créer l'image de base pour votre backend**


```dockerfile
FROM registry.access.redhat.com/ubi8/ubi-minimal@sha256:c6592eb9cdd7ea7fa43beddf507ca2a8c2127f13ef66d49baea2fd28e37f62ba
WORKDIR /work/
RUN chown 1001 /work \
    && chmod "g+rwX" /work \
    && chown 1001:root /work
#COPY --chown=1001:root target/*-runner /work/application
# COPY --chown=1001:root ./src/main/resources/db/migration/ /work/migration
COPY target/*-runner /work/application
RUN chmod 775 /work
EXPOSE 8080
USER 1001
CMD ["./application", "-Dquarkus.http.host=0.0.0.0"]
```



```bash
docker build -f src/main/docker/Dockerfile.arm64 -t barais/correctexam-back:manifest-arm64v8 --build-arg ARCH=arm64v8/ .
```

#### Construire la partie frontend

Clonez le dépôt du frontend. 

:::{attention}
⚠️ mettre à jour webpack/environment.js avec votre nom de domaine.
:::


```bash
git clone https://github.com/correctexam/corrigeExamFront
cd corrigeExamFront
# update webpack/environment.js with your domain name
sudo docker build -f src/main/docker/Dockerfile.arm64 -t barais/correctexam-front::manifest-arm64v8 --build-arg ARCH=arm64v8/ .
# OR using buildx
sudo docker buildx build  -f src/main/docker/Dockerfile --push --platform linux/arm64,linux/amd64  --tag barais/correctexam-front .
```

Vous obtiendrez un nginx avec seulement js, html et js. Vous devez monter la configuration si vous voulez gérer le proxy vers les routes du backend. Je préfère utiliser un nginx bunkerisé pour le routage. 



### Déployer sur votre raspberry 4

Vous pouvez pousser votre image construite sur dockerhub (mettre à jour l'image docker dans le compose docker) et simplement déployer le compose docker sur votre propre raspberry avec les fichiers de configuration nginx.

```yaml
version: '2'
services:
  correctexam-back:
    image: barais/correctexam-back::manifest-arm64v8
    volumes:
# Path for 
      - /tmp/files:/tmp/files:rw 
    restart: always
    ports:
      - 8080:8080
# All quarkus configuration parameters (knobs) could be override through command line. You could also use different options to update the configuration parameters.  https://quarkus.io/guides/config-reference
    command: ./application -Dquarkus.http.host=0.0.0.0 -Dquarkus.datasource.username=root -Dquarkus.datasource.password='' -Dquarkus.datasource.jdbc.url=jdbc:mysql://correctexam-mysql:3306/correctexam?useUnicode=true&characterEncoding=utf8&useSSL=false -Dquarkus.http.cors=true -Dquarkus.http.cors.origins=https://correctexam.github.io -Dquarkus.http.cors.methods=GET,PUT,POST,DELETE,PATCH,OPTIONS -Dquarkus.http.cors.headers=accept,origin,authorization,content-type,x-requested-with -Dquarkus.http.cors.exposed-headers=Content-Disposition -Dquarkus.http.cors.access-control-max-age=24H -Dquarkus.mailer.from=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.host=partage.univ-rennes1.fr -Dquarkus.mailer.port=587 -Dquarkus.mailer.ssl=false -Dquarkus.mailer.username=olivier.barais@univ-rennes1.fr -Dquarkus.mailer.password=TOCHANGE -Djhipster.mail.base-url=https://correctexam.github.io/corrigeExamFront
  correctexam-mysql:
    image: mysql:8.0.20
    volumes:
      - ./../resources/db/migration/:/docker-entrypoint-initdb.d
    environment:
      - MYSQL_USER=root
      - MYSQL_ALLOW_EMPTY_PASSWORD=yes
      - MYSQL_DATABASE=correctexam
    command: mysqld --lower_case_table_names=1 --skip-ssl --character_set_server=utf8mb4 --explicit_defaults_for_timestamp
#    ports:
#      - 3308:3306
  front:
    image: barais/correctexam-front:manifest-arm64v8
#    ports:
#      - 90:80
    volumes:
      -  ./exampleconf/exam.conf:/etc/nginx/conf.d/exam.conf
      -  ./exampleconf/nginx.conf:/etc/nginx/nginx.conf:ro
```


**exam.conf** et **nginx.conf** pourraient ressembler à quelque chose comme cela (vous devez mettre à jour le nom du serveur)


**exam.conf**


```nginx
server {
    listen       80;
    listen  [::]:80;
    # server name to change based on your own domain name for your front
    server_name  correctexam.barais.fr;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location /api {
        proxy_pass http://correctexam-back:8080/api;
        proxy_set_header Host $http_host;

    }

    location /api {
        include proxy_params;
    	proxy_pass http://correctexam-back:8080/api;
    }

    location /management {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/management;
    }

    location /swagger-ui {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/swagger-ui;
    }

    location /v3/api-docs {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/v3/api-docs;
    }

    location /auth {
	include proxy_params;
        proxy_pass http://correctexam-back:8080/auth;

    }

    location /health {
        include proxy_params;
        proxy_pass http://correctexam-back:8080/health;
    }

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
        try_files $uri $uri/ /index.html?$args;

    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

**nginx.conf**

```nginx
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 1000s;
	types_hash_max_size 2048;


    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    gzip  on;
	client_max_body_size 900M;
	client_body_buffer_size     900M;

    include /etc/nginx/conf.d/*.conf;
}
```

## Créer une version sur docker hub

Ensuite, pour la partie frontale, vous pouvez utiliser dockerX ou créer votre image docker pour les différentes architectures ciblées.

```bash
docker manifest create \
barais/correctexam-front:manifest-v1 \
--amend barais/correctexam-front:manifest-amd64 \
--amend barais/correctexam-front:manifest-arm64v8
docker manifest push barais/correctexam-front:manifest-v1
```

Pour le back, créez votre image docker pour les différentes architectures ciblées.

```bash
docker manifest create \
barais/correctexam-back:manifest-v1 \
--amend barais/correctexam-back:manifest-amd64 \
--amend barais/correctexam-back:manifest-arm64v8
docker manifest push barais/correctexam-back:manifest-v1

```
