# Prérequis

Cette stack Drupal tourne sous Docker.
La documentation ci-dessous vous propose de le faire [avec WSL](#documentation-en-utilisant-wsl) ou [sans WSL](#documentation-sans-utiliser-wsl).

# Documentation (en utilisant WSL)

* Docker4drupal => https://wodby.com/docs/stacks/drupal/containers/
* Installer wsl => https://docs.microsoft.com/fr-fr/windows/wsl/install-win10
* Installer docker dans wsl (et changer `/mnt` en `/`) => [Voir cet article](https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly#install-docker-and-docker-compose-within-wsl)

## Get started

### Configuration

La configuration de la stack est définie dans le fichier `.env`

* Si vous déployez sur une machine qui possède déjà un naming DNS, modifiez le paramètre `PROJECT_BASE_URL=drupal9.local`

### Lancer les conteneurs :

```
 make up
```

### Télécharger Drupal dans le conteneur :

```
 make composer install
```

### Installer drupal

```
 make drush 'si --db-url=mysql://drupal:drupal@mariadb:3306/drupal standard -y'
```

### Mettre à jour votre fichier hosts
Dans le fichier `C:\Windows\System32\drivers\etc\hostss`

Ajouter la ligne :
```
127.0.0.1 drupal9.local
```

### Accéder à http://drupal9.local/

# Documentation (sans utiliser WSL)

* Docker4drupal => https://wodby.com/docs/stacks/drupal/containers/
* DockerDestop => https://docs.docker.com/docker-for-windows/install/

## Get started

### Lancer les conteneurs :

```
 docker-compose up -d
```

### Télécharger Drupal dans le conteneur :

```
 docker-compose exec php composer install
```

### Installer drupal

```
 docker-compose exec php drush si --db-url=mysql://drupal:drupal@mariadb:3306/drupal standard -y
```

### Mettre à jour votre fichier hosts
Dans le fichier `C:\Windows\System32\drivers\etc\hostss`

Ajouter la ligne :
```
127.0.0.1 drupal9.local
```

### Accéder à http://drupal9.local/

# Debug

Télécharger Drupal coté windows pour le mapping :

```
composer install
```

Et suivre les consignes de https://wodby.com/docs/stacks/php/local/#ide-configuration
* Mettre `local` en nom de serveur
* Faire le mapping sur `/var/www/html/web`

# Pre-commit

Lors de l'installation des dépendances de développement avec Composer, un pre-commit est automatiquement activé
afin de s'assurer du respect des normes et bonnes pratiques de développement de Drupal.

Cela a pour but de normer une partie des développements afin de gagner en lisibilité du code.

Si vous utilisez Git en ligne de commande, il peut être judicieux de désactiver ce pre-commit (lors d'un merge
ou d'un rebase par exemple). Vous pouvez alors désactiver temporairement le pre-commit en ajoutant
l'option "-n" ou "--no-verify" :

```sh
git commit -nm "Mon message de commit"
git commit -m "Mon message de commit" --no-verify
```

## PHPCS

Il est possible d'exécuter manuellement PHPCS à l'aide de la commande suivante :

```sh
phpcs --standard=Drupal --extensions=php,module,inc,install,test,profile,theme,css,info,txt,md,yml --ignore=node_modules,bower_components,vendor web/modules/custom web/profiles/custom web/themes/custom
```

## ESlint
Les linters JS n'étant pas inclus dans le precommit, vous pouvez les tester manuellement avant de commit.

```sh
npm install
npx web/modules/custom/
npx web/themes/custom/
```


# Drupal en multi-site

## Un site = une base de données

La suite Docker4Drupal fonctionne par défaut en mode mono-site (avec une seule base de données).
Il faut réaliser des modifications manuelles pour donner les droits à l'utiliseur de manipaler la nouvelle base de données.

Remplacer `<database_name>` par le nom de la base données que vous souhaitez créer et exécuter les commandes suivantes :
````sh
docker-compose exec mariadb sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "CREATE DATABASE <database_name>;"'
docker-compose exec mariadb sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "GRANT ALL PRIVILEGES ON <database_name>.*  TO '"'"'${MYSQL_USER}'"'"'@'"'"'%'"'"';"'
````

## Installation

### Site vierge

````sh
docker-compose exec php sh -c 'drush si --sites-subdir="<site_name>" --db-url="mysql://drupal:drupal@mariadb:3306/<database_name>" minimal -y'
````

Drupal va automatiquement crée vos fichiers de configuration dans `web/sites/<site_name>/files/config_XXX/sync`.
Il est conseillé de les déplacer ailleurs car ce dossier est ignoré.
Par exemple : `web/sites/<sites_name>/config/sync`.

Il faut alors adapter en conséquence la déclaration du dossier des configurations dans le fichier "settings.php" de votre nouveau site :
````php
$settings['config_sync_directory'] = $app_root . '/' . $site_path . '/config/sync';
````

### Avec un fichier "settings.php" existant

````sh
docker-compose exec php sh -c 'drush si --sites-subdir="<site_name>" minimal -y'
````

Même remarque que pour un site vierge, il est vivement conseillé de déplacer les configurations dans un autre dossier.

### Avec une configuration existante

````sh
docker-compose exec php sh -c 'drush si --sites-subdir="<site_name>" --existing-config -y'
````

## Gestion des sites

Pour un fonctionnement en multi-site, il faut donner à Drupal la connaissance des hosts par sites.

Il faut alors créer le fichier `web/sites/sites.php` s'il n'existe pas et y insérer une ligne par site par environnement.

Exemples :
````php
$sites['example.localhost'] = 'site1';
$sites['example2.localhost'] = 'site2';
$sites['site3.example.localhost'] = 'site3';
$sites['example2.localhost.site4'] = 'site4';
````
Où
* `site1` est le site principal
* `site2` est un autre site principal
* `site3` est un sous-domaine de `site1`
* `site4` est un site dans un sous-répertoire de `site2`

### Multisite en sous-domaine

Pensez à mettre à jour votre votre `docker-compose.yml` pour que Traefik puisse gérer les sous domaines :
Pour Apache :
````yaml
  apache:
    ...
    labels:
      - 'traefik.backend=${PROJECT_NAME}_apache'
      - 'traefik.port=80'
      - 'traefik.frontend.rule=HostRegexp:{subdomain:[a-z]+}.${PROJECT_BASE_URL}'
````
Pour NGINX :
````yaml
  nginx:
  ...
    labels:
      - 'traefik.backend=${PROJECT_NAME}_nginx'
      - 'traefik.port=80'
      - 'traefik.frontend.rule=HostRegexp:{subdomain:[a-z]+}.${PROJECT_BASE_URL}'
````

### Multisite en domaine dédié

Pensez à mettre à jour votre votre `docker-compose.yml` pour que Traefik puisse gérer les sous domaines :
Pour Apache :
````yaml
  apache:
    ...
    labels:
      - 'traefik.backend=${PROJECT_NAME}_apache'
      - 'traefik.port=80'
      - 'traefik.frontend.rule=${PROJECT_BASE_URL},example2.com'
````
Pour NGINX :
````yaml
  nginx:
  ...
    labels:
      - 'traefik.backend=${PROJECT_NAME}_nginx'
      - 'traefik.port=80'
      - 'traefik.frontend.rule=${PROJECT_BASE_URL},example2.com'
````

### Multisite en sous-répertoire

Il est également possible de faire du multisite en sous-répertoire mais cela demande un peu plus de modifications.

Dans le `docker-compose.yml` ajouter les informations suivantes pour NGINX :
````yaml
  nginx:
  ...
    environment:
      ...
      NGINX_SERVER_EXTRA_CONF_FILEPATH: /etc/nginx/custom/subdirectories.conf
  ...
    volumes:
      ...
      - ./docker/nginx:/etc/nginx/custom
````

A la racine du projet, créer le fichier avec l'arborescence suivante `docker/nginx/subdirectories.conf`
et copier/collez le contenu ci-dessous en remplaçant `<site>` par le nom de votre site.
````text
location ~ ^/<site>/(.*)$ {
    try_files $uri @<site>;
}

location @<site> {
    include fastcgi.conf;
    fastcgi_param QUERY_STRING $query_string;
    fastcgi_param SCRIPT_NAME /<site>/index.php;
    fastcgi_param SCRIPT_FILENAME $document_root/index.php;
    fastcgi_pass php;
    track_uploads uploads 60s;
}
````

Et pour finir, il vous faut créer un lien symbolique dans votre image docker :
````sh
ln -s /var/www/html/web /var/www/html/web/<site>
````

Il ne vous reste plus qu'à mettre à jour votre conteneur NGINX et de vérifier les logs :
````sh
docker-compose up -d nginx && docker-compose logs -f nginx
````

## Commandes drush

Pour executer des commandes drush dans un contexte multisite il faut utiliser l'option `-l` :
```sh
drush -l <site_name> status
```
Ou se placer dans le dossier du site :
```sh
cd /var/www/html/web/sites/<site_name>
drush status
```
