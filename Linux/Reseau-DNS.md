# Linux — réseau et DNS (Debian)

## Vérifier la configuration

~~~bash
ip -br addr
ip addr
ip route
ip link
hostname -I
ping -c 4 192.168.10.1
ping -c 4 8.8.8.8
ping -c 4 example.org
~~~

`ip route` : repérer la **route par défaut**. `ens18` / `eth0` sont des exemples de noms d'interface.

## IP statique (si le système utilise ifupdown)

~~~bash
sudo nano /etc/network/interfaces
~~~

Exemple à adapter :

~~~text
auto ens18
iface ens18 inet static
    address 192.168.10.20
    netmask 255.255.255.0
    gateway 192.168.10.1
~~~

~~~bash
sudo systemctl restart networking
ip addr show ens18
ip route
~~~

Attention : un redémarrage réseau peut **couper la session SSH**. Si le serveur utilise NetworkManager ou systemd-networkd, ne pas appliquer aveuglément cette méthode.

## DNS et ordre de résolution

~~~bash
cat /etc/resolv.conf
cat /etc/nsswitch.conf
getent hosts example.org
~~~

Exemple de ligne dans `/etc/nsswitch.conf` :

~~~text
hosts: files dns
~~~

`files` = `/etc/hosts` ; `dns` = résolution par les serveurs DNS. Le fichier peut contenir d'autres mécanismes selon la machine.

Exemple d'entrée dans `/etc/hosts` :

~~~text
192.168.10.20 web.lab.local web
~~~

Exemple de ligne de serveur DNS :

~~~text
nameserver 192.168.10.10
~~~

Attention : si `/etc/resolv.conf` est géré automatiquement (DHCP, systemd-resolved…), l'éditer à la main peut être temporaire.

## Vérifier la résolution et les ports

~~~bash
getent hosts web.lab.local
nslookup web.lab.local
ss -lntp
~~~

`nslookup` peut nécessiter `dnsutils`. `getent` respecte la configuration de résolution du système.

## Route temporaire si nécessaire

~~~bash
sudo ip route add default via 192.168.10.1
~~~

Ne fonctionne que si la passerelle est joignable ; la commande n'est pas persistante.
