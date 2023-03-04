# Préambule

GLPI (sigle de Gestionnaire Libre de Parc Informatique) est un logiciel
libre de gestion des services informatiques (ITSM) et de gestion des
services d’assistance (issue tracking system et ServiceDesk). Cette
solution libre est éditée en PHP et distribuée sous licence GPL. En tant
que logiciel de gestion des services informatiques (ITSM), les
principaux caractéristiques de GLPI sont les suivantes :

- Gestion multi-entité ;

- Gestion et support multilingue (45 langues disponibles) ;

- Support multi-utilisateur et système d’authentification multiple ;

- Gestion administrative et financière ;

- Fonctionnalités d’inventaire ;

- Gestion d’émission de tickets et des requêtes, fonctionnalités de
contrôle (monitoring) ;

- Gestion des problèmes et des changements ;

- Gestion des licences (ITIL Compliant) ;

- Assignation des équipements : lieu, utilisateurs et groupes ;

- Interface simplifiée permettant aux utilisateurs finaux de soumettre
un ticket ;

-   Générateur de rapports d’actifs et d’assistance : matériel, réseau
    ou interventions (support).

(Source Wikipédia)

# Prérequis

-   RHEL 9

-   Package d’installation GLPI (glpi-10)

# Préparation du système

## Mise à jour du système

De manière à pouvoir installer tous les packages nécessaires, veuillez
bien à effectuer la commande suivante :

    sudo dnf update

Notez bien que cette méthode fonctionnera seulement si vous avez
correctement monté les dépots sur votre système

## Installation et lancement du serveur Web et de la base de donnée

Vous allez devoir utiliser un serveur web et une base de données. Pour
notre cas, il s’agira de :

-   Apache

-   Mariadb

Lancez la commande suivante :

    sudo dnf install httpd mariadb-server -y

![width=50%](Images\1.png)

![width=50%](Images\2.png)

Suite à cela, nous allons activer les services.

    sudo systemctl enable httpd
    sudo systemctl enable mariadb

![width=50%](Images\3.png)

![width=50%](Images\4.png)

Puis nous allons lancer les deux services.

    sudo systemctl start httpd
    sudo systemctl start mariadb

![width=50%](Images\5.png)

![width=50%](Images\6.png)

## Installation de PHP

Nous allons installer tous les paquets nécessaires pour PHP. Pour cela,
tapez la commande suivante :

    sudo dnf install php*

![width=50%](Images\7.png)

Acceptez les conditions en tapant kbd:\[o\].

![width=50%](Images\8.png)

![width=50%](Images\9.png)

## Création et configuration de la base de donnée

Nous allons devoir créer une base de donnée. Pour cela, pour vous
connectez sur SQL, tapez la commande suivante :

    sudo mysql -u root -p

MariaDB devrait s’afficher.

![width=50%](Images\10.png)

### Création de la base

    CREATE DATABASE glpidb;

![width=50%](Images\11.png)

### Création de l’utilisateur de la base de donnée

Nous allons créer un compte spécifique à la base de donnée. Pour cela :

    GRANT ALL ON  glpidb.* TO 'glpi_user'@'localhost' IDENTIFIED BY 'motdepasse';

![width=50%](Images\12.png)

### Droit Utilisateur

Puis nous allons donner des droits.

    FLUSH PRIVILEGES;

![width=50%](Images\13.png)

Quittez en tapant la commande suivante :

    EXIT;

![width=50%](Images\14.png)

# Installation et configuration de GLPI

Récupérez le package GLPI. Il est disponible à l’URL suivant :

<https://github.com/glpi-project/glpi/releases/download/10.0.0/glpi-10.0.0.tgz>

## Décompression

Décompressez le package :

    sudo tar -xvf chemindufichier.tgz /var/www/html/

![width=100%](Images\15.png)

![width=50%](Images\16.png)

## Attribution des droits

Nous allons attribuer un groupe et des droits aux fichiers décompressés.

    sudo chown -R apache:apache /var/www/html/glpi

![width=50%](Images\17.png)

    sudo chmod -R 755 /var/www/html/glpi

![width=50%](Images\18.png)

## Configuration du serveur web

Nous allons configurer le répertoire web de GLPI. Pour cela,
dirigez-vous à l’endroit suivant :

    sudo vim /etc/httpd/conf.d/glpi.conf

![width=50%](Images\19.png)

Puis renseigner les éléments suivants :

    <VirtualHost *:80>
       ServerName server-IP or FQDN
       DocumentRoot /var/www/html/glpi

       ErrorLog "/var/log/httpd/glpi_error.log"
       CustomLog "/var/log/httpd/glpi_access.log" combined

       <Directory> /var/www/html/glpi/config>
               AllowOverride None
               Require all denied
       </Directory>

       <Directory> /var/www/html/glpi/files>
               AllowOverride None
               Require all denied
       </Directory>
    </VirtualHost>

## Stratégies SELinux

Nous allons rajouter des "contextes" dans la politique de contrôle
d’accès de RedHat.

    sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/glpi(/.*)?"
    sudo restorecon -Rv /var/www/html/glpi

![width=100%](Images\20.png)

![width=100%](Images\21.png)

Redémarrez le service Web afin de prendre en compte les modifications.

    sudo systemctl restart httpd

![width=50%](Images\22.png)

## Régle Pare-feu

Nous allons devoir ouvrir le pare-feu pour autoriser HTTP. Pour cela :

    firewall-cmd --zone=public --permanent --add-service=http

![width=100%](Images\23.png)

# Finalisation GLPI

## Langage

La configuration initiale de GLPI n’est pas terminé. Il faut maintenant
lancer un navigateur web (de préférence Firefox) sur votre réseau et
taper l’IP de votre serveur GLPI.

Vous devriez avoir l’image ci-dessous.

Choisissez btn:\[Français\] puis cliquez sur kbd:\[Ok\].

![width=100%](Images\24.png)

## Licence

Acceptez la licence puis cliquez sur kbd:\[Continuer\].

![width=100%](Images\25.png)

## Début de la "vrai" installation

Vous allez choisir d’installer ou mettre à jour GLPI.

Cliquez sur kbd:\[Installer\].

![width=100%](Images\26.png)

## Liste des prérequis manquants

L’installation va effectuer une vérification des pré-requis présents sur
votre système.

Vous pouvez voir dans notre cas, que certains pré-requis sont "suggérés"
mais n’empéchent pas l’installation.

![width=100%](Images\27.png)

![width=100%](Images\28.png)

Vous pourrez revenir ultérieurement là dessus.

Cliquez sur kbd:\[Continuer\].

## Configuration de la connexion à la base de donnée

Nous avons besoin de certaines informations pour connecter la base de
donnée. renseignez les informations que vous avez utiliser pour MariaDB.

![width=100%](Images\29.png)

Puis cliquez sur kbd:\[Continuer\].

## Test de connexion de la BDD

Nous allons tester la connexion de la BDD. Cliquez donc sur
btn:\[glpidb\] et ensuite sur kbd:\[Continuer\].

![width=100%](Images\30.png)

## Initialisation de la BDD

Une initialisation va être effectué. Attendez que ce soit OK.

![width=100%](Images\31.png)

![width=100%](Images\32.png)

Puis cliquez sur kbd:\[Continuer\].

## Statistiques

Cochez ou décochez. Faites comme vous voulez.

![width=100%](Images\33.png)

Cliquez sur kbd:\[Continuer\].

## Etape 5

GLPI est du OpenSource.

![width=100%](Images\34.png)

Cliquez sur kbd:\[Continuer\].

## Etape 6

L’installation est terminé. Bien joué.

Cliquez sur kbd:\[Utiliser GLPI\].

![width=100%](Images\35.png)

# Administration de GLPI

## Problèmes de sécurité

Vous êtes maintenant sur GLPI. Vous pouvez remarquer qu’il vous informe
de problèmes de sécurité. Nous allons corriger cela.

![width=100%](Images\36.png)

Cliquez sur l’utilisateur GLPI et modfifiez le mot de passe.

![width=100%](Images\37.png)

Pensez à valider la modification du mot de passe en cliquant sur
kbd:\[Sauvegarder\].

Effectuez cette modification sur l’ensemble des utilisateurs déja
présents.

Enfin, nous allons supprimer le fichier.

![width=100%](Images\38.png)

Pour cela, lancez la commande :

    sudo rm /var/www/html/glpi/install/install.php

![width=100%](Images\39.png)

Voilà, vous n’avez plus de messages.

![width=100%](Images\40.png)

## Inventaire des postes

Comme vous avez pu le constater, nous n’avons aucune remontés
d’informations. Nous allons devoir installer le agent GLPI sur nos
ordinateurs.

Pour cela, btn:\[créez\] un dossier sur un serveur où vous voulez
partager l’agent. Renseignez les droits suivants.

![width=50%](Images\41.png)

Créez ensuite une GPO et dirigez-vous à l’endroit ci-dessous :

Configuration Ordinateur &gt; Paramètre Windows &gt; Scripts
(démarrage/arrêt) &gt; Démarrage

Ajoutez la ligne :

    msiexec.exe

![width=100%](Images\42.png)

Ajoutez dans Paramètre du script les informations suivantes :

/quiet /i "\\Le chemin réseau" RUNNOW=1
SERVER=”http://mon-serveur-glpi/front/inventory.php”

Comme l’exemple ci-dessous :

![width=100%](Images\44.png)

Appliquez votre GPO sur les ordinateurs. Vous pourrez voir que cela
remonte sur votre console GLPI.

![width=100%](Images\43.png)

# Annexes

Liens des sites utilisés :

<https://fr.techtribune.net/linux/comment-installer-glpi-it-asset-management-sur-les-systemes-rhel/319955/>

<https://neptunet.fr/deploy-agents-for-glpi/>

<https://github.com/glpi-project/glpi/releases/>
