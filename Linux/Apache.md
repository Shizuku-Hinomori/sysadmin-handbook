# Apache — publier un site avec VirtualHost (Debian)

> Exemple fictif : site `/var/www/examen`, serveur accessible par `http://IP_DU_SERVEUR/`, port `80` (ou `8080` si demandé). Méthode avec **nouveau fichier de configuration**, sans réutiliser `000-default.conf`.

## 0. Avec quel compte faire les manipulations ?

**Les commandes d'administration se font sur la VM Debian :**
- soit connecté en **`root`** dans la console : exécuter les commandes **sans** `sudo` ;
- soit connecté avec un **utilisateur disposant de `sudo`** : garder les `sudo` indiqués ci-dessous.

~~~bash
whoami
id
~~~

**Pourquoi ?** Installer un paquet, écrire dans `/etc/apache2/` ou `/var/www/` et recharger un service demandent généralement les droits administrateur.

**Apache ne sert pas le site en tant que root :** sur Debian, les processus qui répondent aux requêtes fonctionnent normalement sous le compte technique **`www-data`**. Ce compte doit pouvoir **lire les fichiers** et **traverser les dossiers** ; inutile de lui donner la propriété du site pour une simple page statique.

## 1. Installer et démarrer Apache

Sur le serveur :

~~~bash
sudo apt update
sudo apt install apache2
sudo systemctl enable --now apache2
systemctl status apache2
~~~

**Pourquoi ces commandes ?**
1. `apt update` actualise la liste des paquets disponibles.
2. `apt install apache2` installe le serveur HTTP.
3. `enable --now` active le service **au démarrage** et **le démarre immédiatement**.
4. `status` vérifie qu'Apache est actif.

Test initial : ouvrir `http://IP_DU_SERVEUR/` dans un navigateur. Une page Apache par défaut peut apparaître.

## 2. Créer le dossier du site et sa page HTML

Toujours depuis **root ou un compte sudo** :

~~~bash
sudo mkdir -p /var/www/examen
sudo nano /var/www/examen/index.html
~~~

Contenu de test :

~~~html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Site examen</title>
</head>
<body>
  <h1>Site Apache opérationnel</h1>
  <img src="image.png" alt="Image de test">
</body>
</html>
~~~

**Pourquoi ?**
- `/var/www/examen` contient les **fichiers du nouveau site**.
- `index.html` est la page que le navigateur affiche par défaut quand on visite la racine du site.
- `src="image.png"` signifie : rechercher `image.png` **dans le même dossier que la page HTML**.

Si une image est fournie, la transférer dans `/var/www/examen/image.png`. Avec MobaXterm, on peut la déposer par SFTP dans le répertoire personnel de l'utilisateur, puis la déplacer/copier avec les droits administrateur :

~~~bash
sudo cp /home/tech/image.png /var/www/examen/image.png
~~~

Adapter `tech` et les noms des fichiers au sujet.

## 3. Donner les permissions avec des **chiffres**

~~~bash
sudo chmod 755 /var/www/examen
sudo chmod 644 /var/www/examen/index.html
sudo chmod 644 /var/www/examen/image.png
ls -ld /var/www/examen
ls -l /var/www/examen
~~~

> N'exécuter la ligne pour `image.png` **que si l'image existe**.

| Permission | À quoi ça sert ? |
|---|---|
| `755` sur le dossier | Propriétaire : lire/écrire/traverser. Groupe et autres : lire/traverser. Apache peut accéder aux fichiers à l'intérieur. |
| `644` sur `index.html` | Propriétaire : lire/écrire. Autres, dont `www-data` : lire. |
| `644` sur `image.png` | Apache peut lire l'image et l'envoyer au navigateur. |

**Pourquoi pas `chmod -R 755` ?** Cela donnerait un droit d'exécution inutile à tous les fichiers. On utilise `755` pour les répertoires et `644` pour les pages/images statiques. Les dossiers parents (comme `/var` et `/var/www`) doivent également être traversables par `www-data`.

## 4. Créer un VirtualHost spécifique

~~~bash
sudo nano /etc/apache2/sites-available/examen.conf
~~~

Ajouter :

~~~apache
<VirtualHost *:80>
    DocumentRoot /var/www/examen

    <Directory /var/www/examen>
        Require all granted
    </Directory>

    ErrorLog /var/log/apache2/examen-error.log
    CustomLog /var/log/apache2/examen-access.log combined
</VirtualHost>
~~~

**À quoi servent les lignes ?**
1. `<VirtualHost *:80>` : définit un site sur le port **80** (HTTP).
2. `DocumentRoot` : indique **où Apache doit chercher `index.html`**.
3. `<Directory>` + `Require all granted` : autorise l'accès HTTP à ce dossier selon la configuration Apache.
4. `ErrorLog` : journal des erreurs du site.
5. `CustomLog` : journal des requêtes reçues.

**Pourquoi créer `examen.conf` ?** Le site garde une configuration indépendante, sans modifier le VirtualHost par défaut `000-default.conf`. Le nom avant `.conf` est libre.

## 5. Activer le site et appliquer la configuration

~~~bash
sudo a2ensite examen.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
~~~

**Pourquoi ?**
- `a2ensite` : active notre fichier `examen.conf` (lien vers `sites-enabled`).
- `a2dissite 000-default.conf` : désactive la page par défaut **si elle n'est plus nécessaire**, pour que le nouveau site réponde à la requête par IP sur le port 80.
- `apache2ctl configtest` : contrôle la syntaxe (**Syntax OK** attendu).
- `reload` : prend en compte la nouvelle configuration.

**Important :** si le sujet demande de conserver d'autres sites, **ne pas désactiver** le VirtualHost par défaut sans raison. Et si `configtest` signale une erreur, corriger **avant** le rechargement.

## 6. Vérifier que le site fonctionne

Sur le serveur :

~~~bash
systemctl status apache2
ss -lntp | grep ':80'
curl -I http://127.0.0.1/
~~~

Sur le PC Windows, ouvrir :

~~~text
http://IP_DU_SERVEUR/
~~~

**Résultat attendu :** la page avec le titre « Site Apache opérationnel » et l'image (si elle a été fournie).

## 7. Changer le port seulement si demandé (exemple : **8080**)

### A. Déclarer le port d'écoute

~~~bash
sudo nano /etc/apache2/ports.conf
~~~

Ajouter, sans dupliquer une ligne existante :

~~~apache
Listen 8080
~~~

**Pourquoi ?** Sans `Listen 8080`, Apache peut avoir un VirtualHost sur 8080 **sans écouter** ce port.

### B. Modifier uniquement le VirtualHost du site

~~~bash
sudo nano /etc/apache2/sites-available/examen.conf
~~~

Remplacer sa première ligne par :

~~~apache
<VirtualHost *:8080>
~~~

**Pourquoi ?** Le site doit correspondre au port déclaré dans `ports.conf`.

### C. Appliquer et tester

~~~bash
sudo apache2ctl configtest
sudo systemctl reload apache2
ss -lntp | grep ':8080'
~~~

Depuis le navigateur :

~~~text
http://IP_DU_SERVEUR:8080/
~~~

Si nécessaire, autoriser **8080/TCP** dans le pare-feu. Ne pas supprimer `Listen 80` si un autre site en a besoin.

## 8. Journaux et preuves pour l'épreuve

~~~bash
sudo journalctl -u apache2 -n 30 --no-pager
sudo tail -n 30 /var/log/apache2/error.log
~~~

Captures possibles : `examen.conf`, `configtest` (« Syntax OK »), `systemctl status apache2` et **page affichée dans le navigateur**.

### Résumé express

1. Administrer en **root**, ou sous **un compte autorisé à utiliser sudo**.
2. Installer Apache et vérifier son service.
3. Créer `/var/www/examen/index.html` (et copier `image.png`).
4. Mettre **755 au dossier**, **644 aux fichiers** pour qu'Apache puisse les lire.
5. Créer `examen.conf` avec le bon `DocumentRoot`.
6. Activer le VirtualHost, vérifier `Syntax OK`, recharger.
7. Tester depuis un navigateur sur le bon port.
