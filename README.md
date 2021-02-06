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

* Si vous déployez sur une machine qui possède déjà un naming DNS, modifiez le paramètre `PROJECT_BASE_URL=drupal9.localhost`

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
127.0.0.1 drupal9.localhost
```

### Accéder à http://drupal9.localhost/

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
127.0.0.1 drupal9.localhost
```

### Accéder à http://drupal9.localhost/

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

## ESlint
Les linters JS n'étant pas inclus dans le precommit, vous pouvez les tester manuellement avant de commit.

```sh
npm install
npx web/modules/custom/
npx web/themes/custom/
```
