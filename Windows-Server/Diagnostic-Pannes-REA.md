# Diagnostic de pannes — cas pratiques REA Windows Server et Debian

> **Objectif :** partir d'un symptôme, isoler sa cause, corriger **le bon paramètre** et prouver que la réparation fonctionne. Ne pas appliquer toutes les commandes d'un bloc. Adapter noms, IP, domaine et interfaces à l'énoncé.

## Méthode commune : constater → isoler → corriger → retester

1. **Constater :** qui est touché ? un seul utilisateur, un poste, un serveur, tout un réseau ? Depuis quand ?
2. **Isoler :** panne d'IP, de DNS, de service, de compte, de GPO, de droits ou de pare-feu ?
3. **Vérifier avant modification :** noter la valeur existante et l'erreur exacte.
4. **Corriger uniquement** le problème identifié.
5. **Retester depuis le client**, pas seulement depuis le serveur, et conserver une preuve.

## 1. Windows — un poste ne rejoint pas le domaine

**Symptôme :** échec de jonction ou « domaine introuvable ».

**Sur le client, ouvrir CMD :**

~~~cmd
ipconfig /all
ping IP_DU_DC
nslookup entreprise.local
nslookup -type=SRV _ldap._tcp.dc._msdcs.entreprise.local
~~~

**Vérifier :**

- IP et masque adaptés, passerelle si nécessaire ;
- le **DNS préféré du client pointe sur le DNS Active Directory** (souvent le DC), et non un DNS public ;
- le domaine utilisé est **le vrai nom DNS** ; 
- le DC est joignable et les enregistrements SRV sont présents.

**Correction possible :** configurer le bon serveur DNS sur la carte, puis relancer la jonction dans **Paramètres système → Nom de l'ordinateur / domaine** avec un compte autorisé.

**Preuve :** ordinateur visible dans AD, puis ouverture de session avec un compte du domaine.

## 2. Windows — le poste a une adresse APIPA (169.254.x.x)

~~~cmd
ipconfig /all
ipconfig /release
ipconfig /renew
~~~

- Carte en DHCP automatique ? câble/réseau virtuel correct ?
- Dans **DHCP** : service démarré, serveur autorisé en AD si applicable, étendue **active**, baux disponibles.
- Si sous-réseau différent du serveur DHCP : relais DHCP requis sur le routeur/L3.
- Vérifier qu'aucun pare-feu ne bloque les échanges.

**Preuve :** bail obtenu, IP dans la plage attendue, passerelle et DNS corrects. [Voir la fiche DHCP](DHCP.md).

## 3. Windows — le poste a une IP, mais les noms ne se résolvent pas

**Comparer par étapes :**

~~~cmd
ipconfig /all
ping 192.168.10.10
nslookup entreprise.local
nslookup entreprise.local 192.168.10.10
ipconfig /flushdns
~~~

- Si l'IP du serveur répond mais pas son nom : examiner **DNS**, pas DHCP en premier.
- Vérifier le serveur DNS annoncé au client (option **006** DHCP si applicable).
- Sur le serveur DNS : console \`dnsmgmt.msc\` → **Zones de recherche directes** → zone du domaine → enregistrement demandé.
- Si \`nslookup\` marche avec une adresse de serveur explicite seulement, examiner les DNS configurés sur le client.
- Si les noms répondent mais pas \`ping\`, cela ne prouve pas forcément une panne DNS : ICMP peut être bloqué.

**Preuve :** \`nslookup\` retourne l'enregistrement correct depuis le client.

## 4. Windows — une GPO n'est pas appliquée

~~~cmd
whoami
gpupdate /force
gpresult /r
gpresult /h "%TEMP%\rapport-gpo.html"
~~~

**Vérifier :** bonne **OU utilisateur ou ordinateur** selon le paramètre, lien activé, filtrage de sécurité, DNS du domaine, connexion du compte cible. Une stratégie utilisateur peut nécessiter une **déconnexion/reconnexion**.

**Journal :** \`eventvwr.msc\` → **Journaux des applications et services → Microsoft → Windows → GroupPolicy → Operational**.

**Preuve :** GPO présente dans les stratégies appliquées **et** comportement attendu visible. [Voir GPO](GPO.md).

## 5. Windows — « Accès refusé » sur un partage

**Client :**

~~~cmd
whoami /groups
net use
~~~

**PowerShell client :**

~~~powershell
Test-NetConnection SRV-FICHIERS -Port 445
~~~

**Serveur :**

~~~powershell
Get-SmbShare -Name RH
Get-SmbShareAccess -Name RH
Get-Acl "D:\Partages\RH" | Format-List
~~~

**Vérifier :** partage \`\\SRV-FICHIERS\RH\` réel, droits **de partage + NTFS**, groupes **AGDLP**, session utilisateur et jeton de groupes (ouvrir une nouvelle session après ajout à un groupe).

**Preuve :** création/modification d'un fichier de test avec un compte autorisé ; refus avec compte non autorisé si demandé. [Voir Partages SMB](Partages-SMB.md).

## 6. Windows — un compte AD ne peut pas se connecter

**Console :** \`dsa.msc\` → utilisateur → **Propriétés → Compte**.

- Compte **désactivé** ?
- Compte **verrouillé** ? (différent de désactivé)
- Compte **expiré** ? horaires de connexion autorisés ?
- Mot de passe expiré ou changement imposé ?
- Poste joint au domaine et DC accessible ?

**PowerShell (RSAT/AD) :**

~~~powershell
Get-ADUser jdupont -Properties Enabled,LockedOut,AccountExpirationDate,PasswordExpired | Select-Object Name,Enabled,LockedOut,AccountExpirationDate,PasswordExpired
~~~

**Corriger seulement si demandé :** activer, déverrouiller, réinitialiser le mot de passe ou modifier une date. **Preuve :** ouverture de session réussie.

## 7. Debian — pas de réseau ou DNS en panne

**Sur le serveur Debian :**

~~~bash
ip -br addr
ip route
ping -c 3 192.168.10.1
ping -c 3 8.8.8.8
getent hosts example.org
cat /etc/nsswitch.conf
cat /etc/resolv.conf
cat /etc/hosts
~~~

**Lecture rapide :**

| Résultat | Piste |
|---|---|
| Pas de bonne adresse IP | interface, configuration IP, DHCP |
| IP correcte mais pas de route | passerelle / route par défaut |
| IP externe accessible, nom impossible | DNS ou ordre de résolution |
| \`/etc/hosts\` fournit la mauvaise IP | corriger l'entrée locale |
| \`hosts:\` n'inclut pas \`dns\` | contrôler \`nsswitch.conf\` |
| \`resolv.conf\` incorrect | contrôler le gestionnaire réseau et le DNS configuré |

**Attention :** \`/etc/resolv.conf\` peut être régénéré automatiquement. Ne pas redémarrer le réseau depuis SSH sans garder un accès console.

**Preuve :** \`getent hosts\` renvoie la bonne adresse ; \`ping\`/connexion réussissent selon le protocole testé.

## 8. Debian — Apache ne démarre pas ou n'affiche pas le site

~~~bash
systemctl status apache2
sudo apache2ctl configtest
ss -lntp | grep -E ':80|:8080'
sudo journalctl -u apache2 -n 30 --no-pager
sudo tail -n 30 /var/log/apache2/error.log
ls -la /etc/apache2/sites-enabled/
~~~

- **Syntaxe en erreur :** vérifier la ligne du VirtualHost et corriger.
- **Service inactif :** examiner les journaux avant de redémarrer.
- **Mauvaise page :** bon \`DocumentRoot\` ? bon site dans \`sites-enabled\` ? VirtualHost par défaut prioritaire ?
- **Erreur 403 :** droits de lecture/traversée du site, bloc \`<Directory>\`, \`Require all granted\`.
- **Erreur 404 :** chemin du fichier ou nom du document.
- **Port 8080 demandé :** \`Listen 8080\` dans \`ports.conf\` **et** \`<VirtualHost *:8080>\`.

**Après correction :**

~~~bash
sudo apache2ctl configtest
sudo systemctl reload apache2
curl -I http://127.0.0.1/
~~~

Adapter le port à l'énoncé et tester aussi dans le **navigateur du client**. [Voir Apache](../Linux/Apache.md).

## 9. Debian — SSH refuse la connexion ou la clé

**Sur la console du serveur :**

~~~bash
systemctl status ssh
sudo sshd -t
sudo sshd -T | grep -E '^(port|permitrootlogin|pubkeyauthentication|passwordauthentication|kbdinteractiveauthentication) '
ss -lntp | grep ssh
sudo journalctl -u ssh -n 40 --no-pager
~~~

**Vérifier :**

- bonne IP, **bon port** dans MobaXterm ;
- bon utilisateur, présence du fichier \`/home/tech/.ssh/authorized_keys\` **dans son home** ;
- clé **publique** correcte (une ligne), clé **privée** conservée sur le poste client ;
- permissions \`700\` sur \`.ssh\` et \`600\` sur \`authorized_keys\` ; propriétaire correct ;
- \`PermitRootLogin no\` interdit SSH directement en root ;
- ne pas désactiver le mot de passe avant d'avoir validé la connexion par clé.

~~~bash
sudo ls -ld /home/tech /home/tech/.ssh
sudo ls -l /home/tech/.ssh/authorized_keys
sudo chown -R tech:tech /home/tech/.ssh
sudo chmod 700 /home/tech/.ssh
sudo chmod 600 /home/tech/.ssh/authorized_keys
~~~

La commande \`chown -R\` n'est à utiliser que pour le **dossier .ssh de ce compte** et si le propriétaire est incorrect. **Preuve :** nouvelle session MobaXterm réussie. [Voir SSH](../Linux/SSH.md).

## 10. Debian — problème de droits sur un dossier

~~~bash
whoami
id
ls -ld /srv/partage
ls -la /srv/partage
namei -l /srv/partage
~~~

**Distinguer :** propriétaire, groupe, \`r\` lire, \`w\` écrire, \`x\` traverser un répertoire. L'accès dépend aussi des **dossiers parents**.

**Exemple de correction uniquement si demandé :**

~~~bash
sudo chown tech:projet /srv/partage
sudo chmod 770 /srv/partage
~~~

**Preuve :** l'utilisateur autorisé peut créer/lire le fichier requis, et un utilisateur hors groupe n'a pas un accès excessif.

## 11. Résumé : quel test choisir en premier ?

| Symptôme | Test de départ |
|---|---|
| Poste sans IP | \`ipconfig /all\` / \`ip -br addr\` |
| IP OK, nom KO | \`nslookup\` / \`getent hosts\` |
| Domaine Windows introuvable | DNS AD + SRV \`_ldap._tcp.dc._msdcs\` |
| GPO non appliquée | \`gpresult /r\` + GPMC |
| Dossier partagé refusé | \`whoami /groups\` + droits SMB/NTFS |
| Apache KO | \`apache2ctl configtest\` + \`journalctl -u apache2\` |
| SSH KO | \`sshd -t\` + \`journalctl -u ssh\` |
| Droits Linux KO | \`id\`, \`namei -l\`, \`ls -ld\` |

> Pour une REA : montrer **l'état initial**, **la manipulation réalisée**, puis **le test final**. Ne pas installer ni reconfigurer inutilement un service déjà fonctionnel.

**Référence :** [Dépannage GPO — Microsoft Learn](https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/group-policy/applying-group-policy-troubleshooting-guidance)
