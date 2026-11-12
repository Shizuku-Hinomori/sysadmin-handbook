# SSH — connexion, MobaXterm et sécurisation (Debian)

> **Exemples fictifs** : utilisateur `tech`, serveur `192.168.10.20`, port personnalisé `2220`. Remplacer par les données de l'énoncé.

## 1. Identifier le compte utilisé et vérifier SSH sur le serveur

Sur **la VM Debian**, ouvrir la console ou une session déjà fonctionnelle.

- Avec `root`, saisir les commandes **sans `sudo`**.
- Avec un compte administrateur autorisé à utiliser `sudo`, conserver `sudo`.

~~~bash
whoami
systemctl status ssh
sudo apt update
sudo apt install openssh-server
sudo systemctl enable --now ssh
ss -lntp | grep ':22'
~~~

**Pourquoi ?** Vérifier que le serveur SSH est installé, actif et à l'écoute. Sous Debian, l'unité systemd se nomme généralement `ssh` ; le processus est `sshd`.

## 2. Créer l'utilisateur qui se connectera

Sur **le serveur Debian** :

~~~bash
sudo adduser tech
id tech
getent passwd tech
~~~

`adduser` crée normalement le dossier personnel `/home/tech` et demande un mot de passe.

Test initial depuis **le PC client** : ouvrir une session MobaXterm → **Session → SSH**, saisir l'IP `192.168.10.20`, cocher **Specify username**, renseigner `tech` et utiliser le port `22` tant qu'il n'a pas été changé.

## 3. Méthode A — générer les clés dans MobaXterm (interface graphique)

À effectuer **sur le PC Windows (client), pas sur Debian**.

1. Ouvrir **MobaXterm → Tools → MobaKeyGen (SSH key generator)**.
2. Choisir le type **Ed25519** si proposé (sinon RSA avec une taille de clé suffisante), puis cliquer sur **Generate** et déplacer la souris si demandé.
3. Saisir si possible une **passphrase** pour protéger la clé privée.
4. Cliquer sur **Save private key** et conserver le fichier privé `.ppk` **sur le PC Windows**, dans un endroit sûr.
5. **Copier le texte** affiché dans la zone **« Public key for pasting into OpenSSH authorized_keys file »**. Il commence normalement par `ssh-ed25519` ou `ssh-rsa` : c'est la **clé publique** à coller sur Debian. Ne pas copier le texte `.ppk` (clé privée).
6. Sur Debian, préparer le dossier et ouvrir le fichier `authorized_keys` **de l'utilisateur `tech`** :

~~~bash
sudo install -d -m 700 -o tech -g tech /home/tech/.ssh
sudo nano /home/tech/.ssh/authorized_keys
~~~

7. Coller la **clé publique sur une seule ligne**, enregistrer (Nano : **Ctrl+O**, Entrée, **Ctrl+X**), puis appliquer les permissions :

~~~bash
sudo chown tech:tech /home/tech/.ssh/authorized_keys
sudo chmod 700 /home/tech/.ssh
sudo chmod 600 /home/tech/.ssh/authorized_keys
~~~

**Pourquoi ?**
- `authorized_keys` indique au serveur **quelles clés publiques sont autorisées pour ce compte**.
- `700` sur `.ssh` : seul le propriétaire peut y accéder.
- `600` sur `authorized_keys` : seul le propriétaire lit/écrit ce fichier.
- La **clé privée n'est jamais copiée sur le serveur**.

8. Dans MobaXterm, ouvrir/créer **Session → SSH**, indiquer l'IP du serveur et **Specify username = tech**.
9. Aller dans **Advanced SSH settings**, cocher **Use private key** et sélectionner le fichier privé `.ppk`.
10. Tester la connexion. La passphrase de la clé peut être demandée ; **le mot de passe du compte Debian ne doit plus être nécessaire** si la connexion par clé fonctionne correctement.

> Si le serveur demande toujours le mot de passe de `tech`, vérifier le bon compte, la clé publique copiée, les droits et le fichier `authorized_keys`. Ne pas désactiver l'authentification par mot de passe avant d'avoir réussi ce test.

## 4. Méthode B — générer les clés en ligne de commande sur le client

Depuis un terminal client (Linux, ou Windows avec OpenSSH) :

~~~bash
ssh-keygen -t ed25519
~~~

- **Clé privée** : `id_ed25519`, **sur le client**.
- **Clé publique** : `id_ed25519.pub`, à ajouter dans le fichier `/home/tech/.ssh/authorized_keys` **sur le serveur**.

Depuis un **client Linux** :

~~~bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub tech@192.168.10.20
ssh tech@192.168.10.20
~~~

Depuis **PowerShell Windows**, afficher la clé publique pour la copier :

~~~powershell
Get-Content "$env:USERPROFILE\.ssh\id_ed25519.pub"
~~~

Pour préparer le fichier sur le serveur, utiliser les commandes de la **méthode A, étapes 6 et 7**. Avec une clé privée au format OpenSSH, le client Windows peut aussi utiliser la commande `ssh` et l'option `-i` si nécessaire.

## 5. Sécuriser `sshd_config` (sur le serveur Debian)

~~~bash
sudo nano /etc/ssh/sshd_config
~~~

Chercher les directives et définir les valeurs demandées :

~~~text
Port 2220
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
~~~

| Paramètre | Pourquoi ? |
|---|---|
| `Port 2220` | Port personnalisé cité en formation. **Ne remplace pas** une sécurité d'authentification. |
| `PermitRootLogin no` | Bloque la connexion SSH **directe** en root. |
| `PubkeyAuthentication yes` | Autorise l'authentification par clé publique. |
| `PasswordAuthentication no` | Empêche la connexion SSH par mot de passe. |
| `KbdInteractiveAuthentication no` | Désactive aussi le mécanisme interactif pouvant servir à demander un mot de passe. |

Retirer le `#` si la directive est commentée, et vérifier qu'aucune directive active contradictoire n'est présente. Des fichiers dans `/etc/ssh/sshd_config.d/` peuvent aussi être inclus.

**Attention : `PasswordAuthentication no` seulement après un test de connexion par clé réussi.** Ne pas couper la session SSH déjà ouverte avant le test final.

~~~bash
sudo sshd -t
sudo systemctl reload ssh
sudo sshd -T | grep -E '^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
~~~

- `sshd -t` : teste la syntaxe (aucune sortie en cas de succès).
- `reload` : applique la configuration sans redémarrer entièrement le service.
- `sshd -T` : affiche les paramètres réellement pris en compte.

## 6. Tester le **nouveau port** avec MobaXterm

Dans la session **SSH** MobaXterm :

1. **Remote host** : `192.168.10.20`.
2. **Specify username** : `tech`.
3. **Port** : `2220` (si ce paramètre a été modifié).
4. **Advanced SSH settings → Use private key** : sélectionner la clé privée `.ppk`.
5. Ouvrir une **nouvelle connexion**. Vérifier que la connexion réussit **sans mot de passe de compte**.

Si un pare-feu est actif, autoriser le port configuré. Vérifier ensuite que la connexion directe `root` est interdite si c'est demandé.

## 7. Captures utiles pour la réalisation

- Extrait de `sshd_config` montrant les directives modifiées.
- Résultat de `systemctl status ssh` et/ou `sshd -T`.
- Connexion MobaXterm réussie avec le bon utilisateur et le bon port.

**Rappel :** les clés d'hôte générées lors de l'installation du serveur SSH ne remplacent **pas** la clé publique du compte `tech` dans `authorized_keys`.

**Documentation :** [MobaXterm — Documentation officielle](https://mobaxterm.mobatek.net/documentation.html) · [MobaKeyGen — procédure illustrée](https://www.cc.kyushu-u.ac.jp/scp/en/usage/login/create_keypair_mobaxterm/)
