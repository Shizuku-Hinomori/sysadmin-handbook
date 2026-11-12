# Linux — permissions, fichiers et propriétaires

## Afficher

~~~bash
pwd
ls -la
ls -ld /srv/partage
stat /srv/partage
~~~

Droits : **utilisateur / groupe / autres**. `r = 4` (lecture), `w = 2` (écriture), `x = 1` (exécution/traversée du répertoire).

| Mode | Usage fréquent |
|---|---|
| `700` | Propriétaire seul : tout |
| `750` | Propriétaire : tout ; groupe : lire/traverser |
| `755` | Propriétaire : tout ; autres : lire/traverser |
| `644` | Fichier lisible par tous, modifiable par propriétaire |
| `600` | Fichier lisible et modifiable par propriétaire seul |

## Changer les droits

~~~bash
sudo chmod 750 /srv/partage
sudo chmod 600 /home/tech/.ssh/authorized_keys
sudo chmod u+x /srv/script.sh
~~~

Attention à `-R` : appliquer `chmod -R 644` à des répertoires leur retirerait le droit `x`.

## Changer propriétaire et groupe

~~~bash
sudo chown tech:projet /srv/partage
sudo chown -R tech:projet /srv/partage
sudo chgrp projet /srv/partage
~~~

## Fichiers et dossiers

~~~bash
mkdir -p /srv/partage
touch /srv/partage/test.txt
cp fichier.txt /srv/partage/
mv ancien.txt nouveau.txt
find /srv -name '*.txt'
~~~

## SSH : permissions recommandées

~~~bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
~~~
