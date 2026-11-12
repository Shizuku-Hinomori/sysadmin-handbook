# Linux — systemd, journaux et paquets

## Commandes de service

~~~bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl stop ssh
sudo systemctl restart ssh
sudo systemctl reload ssh
sudo systemctl enable ssh
sudo systemctl disable ssh
systemctl is-enabled ssh
systemctl is-active ssh
~~~

Remplacer `ssh` par `apache2`, etc.

- `start` démarre maintenant ; `enable` démarre au prochain boot.
- `restart` relance le service ; `reload` recharge sa configuration, si compatible.

## Journaux

~~~bash
journalctl -u ssh -n 50 --no-pager
sudo journalctl -u apache2 -f
~~~

`-u` : service ciblé ; `-n` : dernières lignes ; `-f` : suivi (Ctrl+C pour quitter).

## Paquets Debian

~~~bash
sudo apt update
sudo apt install apache2
apt list --installed
dpkg -l | grep apache2
~~~

`apt update` met à jour la **liste** des paquets ; `apt upgrade` met à jour les paquets installés. `dpkg` gère les paquets `.deb` locaux.

## Services qui écoutent

~~~bash
ss -lntp
ss -lntup
ps aux | grep apache2
~~~

## Vérifier une réalisation

1. Service actif ?
2. Activé au démarrage si demandé ?
3. Syntaxe du fichier correcte ?
4. Bon port en écoute ?
5. Test réel depuis le client réussi ?
