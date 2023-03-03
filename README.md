# GLPI
:experimental:
:icons: font
:toc: left
:source-highlighter: rouge
:toclevels: 4
:toc-title: Sommaire

= Installation et Configuration de GLPI

toc::[]

<<<

:sectnums:

== Préambule

GLPI (sigle de Gestionnaire Libre de Parc Informatique) est un logiciel libre de gestion des services informatiques (ITSM) et de gestion des services d'assistance (issue tracking system et ServiceDesk). Cette solution libre est éditée en PHP et distribuée sous licence GPL.
En tant que logiciel de gestion des services informatiques (ITSM), les principaux caractéristiques de GLPI sont les suivantes :

- Gestion multi-entité ;

- Gestion et support multilingue (45 langues disponibles) ;

- Support multi-utilisateur et système d'authentification multiple ;

- Gestion administrative et financière ;

- Fonctionnalités d'inventaire ;

- Gestion d’émission de tickets et des requêtes, fonctionnalités de contrôle (monitoring) ;

- Gestion des problèmes et des changements ;

- Gestion des licences (ITIL Compliant) ;

- Assignation des équipements : lieu, utilisateurs et groupes ;

- Interface simplifiée permettant aux utilisateurs finaux de soumettre un ticket ;

- Générateur de rapports d'actifs et d'assistance : matériel, réseau ou interventions (support).

(Source Wikipédia)

== Prérequis

- RHEL 9
- Package d'installation GLPI (glpi-10)

== Préparation du système

=== Mise à jour du système

De manière à pouvoir installer tous les packages nécessaires, veuillez bien à effectuer la commande suivante :

[source,bash]
----
sudo dnf update
----

IMPORTANT: Notez bien que cette méthode fonctionnera seulement si vous avez correctement monté les dépots sur votre système

=== Installation et lancement du serveur Web et de la base de donnée

Vous allez devoir utiliser un serveur web et une base de données. Pour notre cas, il s'agira de :

* Apache
* Mariadb

Lancez la commande suivante :

[source,bash]
----
sudo dnf install httpd mariadb-server -y
----

image::Images\1.png[width=50%]

image::Images\2.png[width=50%]

Suite à cela, nous allons activer les services.

[source,bash]
----
sudo systemctl enable httpd
sudo systemctl enable mariadb
----

image::Images\3.png[width=50%]

image::Images\4.png[width=50%]

Puis nous allons lancer les deux services.

[source,bash]
----
sudo systemctl start httpd
sudo systemctl start mariadb
----

image::Images\5.png[width=50%]

image::Images\6.png[width=50%]

=== Installation de PHP

Nous allons installer tous les paquets nécessaires pour PHP. Pour cela, tapez la commande suivante :

[source,bash]
----
sudo dnf install php*
----

image::Images\7.png[width=50%]

Acceptez les conditions en tapant kbd:[o].

image::Images\8.png[width=50%]

image::Images\9.png[width=50%]

=== Création et configuration de la base de donnée

Nous allons devoir créer une base de donnée. Pour cela, pour vous connectez sur SQL, tapez la commande suivante :

[source,bash]
----
sudo mysql -u root -p
----

MariaDB devrait s'afficher.

image::Images\10.png[width=50%]

==== Création de la base

[source,sql]
----
CREATE DATABASE glpidb;
----

image::Images\11.png[width=50%]

==== Création de l'utilisateur de la base de donnée

Nous allons créer un compte spécifique à la base de donnée. Pour cela :

[source,sql]
----
GRANT ALL ON  glpidb.* TO 'glpi_user'@'localhost' IDENTIFIED BY 'motdepasse';
----

image::Images\12.png[width=50%]

==== Droit Utilisateur

Puis nous allons donner des droits.

[source,sql]
----
FLUSH PRIVILEGES;
----

image::Images\13.png[width=50%]

Quittez en tapant la commande suivante :

[source,sql]
----
EXIT;
----

image::Images\14.png[width=50%]

== Installation et configuration de GLPI

Récupérez le package GLPI. Il est disponible à l'URL suivant :

https://github.com/glpi-project/glpi/releases/download/10.0.0/glpi-10.0.0.tgz

=== Décompression

Décompressez le package :

[source,bash]
----
sudo tar -xvf chemindufichier.tgz /var/www/html/
----

image::Images\15.png[width=100%]

image::Images\16.png[width=50%]

=== Attribution des droits

Nous allons attribuer un groupe et des droits aux fichiers décompressés.

[source,bash]
----
sudo chown -R apache:apache /var/www/html/glpi
----

image::Images\17.png[width=50%]

[source,bash]
----
sudo chmod -R 755 /var/www/html/glpi
----

image::Images\18.png[width=50%]

=== Configuration du serveur web

Nous allons configurer le répertoire web de GLPI. Pour cela, dirigez-vous à l'endroit suivant :

[source,bash]
----
sudo vim /etc/httpd/conf.d/glpi.conf
----

image::Images\19.png[width=50%]

Puis renseigner les éléments suivants :

[source,bash]
----
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
----

=== Stratégies SELinux

Nous allons rajouter des "contextes" dans la politique de contrôle d'accès de RedHat.

[source,bash]
----
sudo semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/glpi(/.*)?"
sudo restorecon -Rv /var/www/html/glpi
----

image::Images\20.png[width=100%]

image::Images\21.png[width=100%]

Redémarrez le service Web afin de prendre en compte les modifications.

[source,bash]
----
sudo systemctl restart httpd
----

image::Images\22.png[width=50%]

=== Régle Pare-feu

Nous allons devoir ouvrir le pare-feu pour autoriser HTTP. Pour cela :

[source,bash]
----
firewall-cmd --zone=public --permanent --add-service=http
----

image::Images\23.png[width=100%]

== Finalisation GLPI

=== Langage

La configuration initiale de GLPI n'est pas terminé. Il faut maintenant lancer  un navigateur web (de préférence Firefox) sur votre réseau et taper l'IP de votre serveur GLPI.

Vous devriez avoir l'image ci-dessous. 

Choisissez btn:[Français] puis cliquez sur kbd:[Ok].

image::Images\24.png[width=100%]

=== Licence 

Acceptez la licence puis cliquez sur kbd:[Continuer].

image::Images\25.png[width=100%]

=== Début de la "vrai" installation

Vous allez choisir d'installer ou mettre à jour GLPI.

Cliquez sur kbd:[Installer].

image::Images\26.png[width=100%]

=== Liste des prérequis manquants

L'installation va effectuer une vérification des pré-requis présents sur votre système.

Vous pouvez voir dans notre cas, que certains pré-requis sont "suggérés" mais n'empéchent pas l'installation.

image::Images\27.png[width=100%]

image::Images\28.png[width=100%]

Vous pourrez revenir ultérieurement là dessus.

Cliquez sur kbd:[Continuer].

=== Configuration de la connexion à la base de donnée

Nous avons besoin de certaines informations pour connecter la base de donnée. renseignez les informations que vous avez utiliser pour MariaDB.

image::Images\29.png[width=100%]

Puis cliquez sur kbd:[Continuer].

=== Test de connexion de la BDD

Nous allons tester la connexion de la BDD. Cliquez donc sur btn:[glpidb] et ensuite sur kbd:[Continuer].

image::Images\30.png[width=100%]

=== Initialisation de la BDD

Une initialisation va être effectué. Attendez que ce soit OK.

image::Images\31.png[width=100%]

image::Images\32.png[width=100%]

Puis cliquez sur kbd:[Continuer].

=== Statistiques

Cochez ou décochez. Faites comme vous voulez.

image::Images\33.png[width=100%]

Cliquez sur kbd:[Continuer].

=== Etape 5

GLPI est du OpenSource.

image::Images\34.png[width=100%]

Cliquez sur kbd:[Continuer].

=== Etape 6 

L'installation est terminé. Bien joué.

Cliquez sur kbd:[Utiliser GLPI].

image::Images\35.png[width=100%]

== Administration de GLPI

=== Problèmes de sécurité

Vous êtes maintenant sur GLPI. Vous pouvez remarquer qu'il vous informe de problèmes de sécurité. Nous allons corriger cela.

image::Images\36.png[width=100%]

Cliquez sur l'utilisateur GLPI et modfifiez le mot de passe.

image::Images\37.png[width=100%]

Pensez à valider la modification du mot de passe en cliquant sur kbd:[Sauvegarder].

NOTE: Effectuez cette modification sur l'ensemble des utilisateurs déja présents.

Enfin, nous allons supprimer le fichier.

image::Images\38.png[width=100%]

Pour cela, lancez la commande :

[source,bash]
----
sudo rm /var/www/html/glpi/install/install.php
----

image::Images\39.png[width=100%]

Voilà, vous n'avez plus de messages.

image::Images\40.png[width=100%]

=== Inventaire des postes

Comme vous avez pu le constater, nous n'avons aucune remontés d'informations. Nous allons devoir installer le agent GLPI sur nos ordinateurs.

Pour cela, btn:[créez] un dossier sur un serveur où vous voulez partager l'agent. Renseignez les droits suivants.

image::Images\41.png[width=50%]

Créez ensuite une GPO et dirigez-vous à l'endroit ci-dessous :

Configuration Ordinateur > Paramètre Windows > Scripts (démarrage/arrêt) > Démarrage

Ajoutez la ligne :

[source,cmd]
----
msiexec.exe
----

image::Images\42.png[width=100%]

Ajoutez dans Paramètre du script les informations suivantes :

/quiet /i "\\Le chemin réseau" RUNNOW=1 SERVER=”http://mon-serveur-glpi/front/inventory.php”

Comme l'exemple ci-dessous :

image::Images\44.png[width=100%]

Appliquez votre GPO sur les ordinateurs. Vous pourrez voir que cela remonte sur votre console GLPI.

image::Images\43.png[width=100%]

== Annexes

Liens des sites utilisés :

https://fr.techtribune.net/linux/comment-installer-glpi-it-asset-management-sur-les-systemes-rhel/319955/

https://neptunet.fr/deploy-agents-for-glpi/

https://github.com/glpi-project/glpi/releases/
