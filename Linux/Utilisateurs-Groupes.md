# Linux — utilisateurs et groupes

## Voir les comptes et groupes

~~~bash
whoami
id
id tech
getent passwd tech
getent group
cat /etc/passwd
~~~

`/etc/passwd` répertorie les comptes ; les hachages des mots de passe se trouvent normalement dans `/etc/shadow`.

## Créer / modifier

~~~bash
sudo adduser tech
sudo passwd tech
sudo usermod -aG sudo tech
id tech
~~~

`usermod -aG` ajoute au groupe secondaire **sans retirer** les autres. `sudo` doit être installé et configuré pour accorder les droits.

Pour un répertoire personnel personnalisé **à la création** :

~~~bash
sudo adduser --home /srv/tech tech
~~~

## Groupes

~~~bash
sudo groupadd projet
sudo usermod -aG projet tech
getent group projet
~~~

Une reconnexion peut être nécessaire pour actualiser les groupes de la session.

## Changer de session

~~~bash
su - tech
sudo -i
~~~

## Supprimer — uniquement si demandé

~~~bash
sudo deluser tech
sudo deluser --remove-home tech
~~~

La deuxième commande supprime le dossier personnel.
