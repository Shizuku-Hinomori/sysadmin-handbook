# DHCP Windows Server — installer, créer une étendue, options et réservation

> Exemple fictif : réseau `192.168.10.0/24`, passerelle `192.168.10.1`, DNS/serveur `192.168.10.10`. Adapter à l'énoncé. Ne pas créer de deuxième DHCP actif sur un réseau de production sans autorisation.

## 1. Installer le rôle si demandé

~~~powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Get-WindowsFeature DHCP
~~~

Après installation dans un domaine AD, effectuer la configuration post-installation (autorisation du serveur, groupes de sécurité si nécessaire) via l'assistant ou les commandes adaptées.

## 2. Autoriser le serveur DHCP dans AD (si serveur membre du domaine)

~~~powershell
Add-DhcpServerInDC -DnsName "srv-dhcp.entreprise.local" -IPAddress 192.168.10.10
Get-DhcpServerInDC
~~~

## 3. Créer et activer une étendue IPv4

~~~powershell
Add-DhcpServerv4Scope -Name "LAN10" -StartRange 192.168.10.100 -EndRange 192.168.10.200 -SubnetMask 255.255.255.0 -State Active
Get-DhcpServerv4Scope
~~~

## 4. Définir les options (passerelle, DNS, suffixe)

~~~powershell
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -Router 192.168.10.1 -DnsServer 192.168.10.10 -DnsDomain "entreprise.local"
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
~~~

## 5. Réserver une adresse

~~~powershell
Add-DhcpServerv4Reservation -ScopeId 192.168.10.0 -IPAddress 192.168.10.120 -ClientId "00-11-22-33-44-55" -Description "Poste-test"
Get-DhcpServerv4Reservation -ScopeId 192.168.10.0
~~~

L'adresse réservée doit être disponible pour l'usage DHCP et le `ClientId` correspondre à l'identifiant transmis par le client (souvent son adresse MAC).

## 6. Interface graphique

`Gestionnaire de serveur → Outils → DHCP → IPv4` → clic droit **Nouvelle étendue** → plage, exclusions, durée de bail, passerelle, DNS et activation.

Pour une réservation : `Étendue → Réservations → Nouvelle réservation`.

## 7. Tester sur le client Windows

~~~cmd
ipconfig /release
ipconfig /renew
ipconfig /all
~~~

Vérifier l'adresse, la passerelle, le DNS, l'identité du serveur DHCP et la portée du bail. Sur d'autres VLAN, un relais DHCP (ip helper) peut être requis.

---

## 8. Cas pratique REA — créer une étendue DHCP entièrement dans la console

**Énoncé type :** pour le réseau `192.168.10.0/24`, distribuer les adresses `192.168.10.100` à `192.168.10.200`, avec passerelle `192.168.10.1` et DNS `192.168.10.10`.

**Préconditions :** le serveur possède une IP fixe ; le rôle DHCP est installé ; le serveur DHCP est **autorisé dans AD**, si applicable. Ne pas créer de DHCP concurrent sur un réseau réel.

**Sur Windows Server :**

1. Ouvrir **Gestionnaire de serveur → Outils → DHCP** (`dhcpmgmt.msc`).
2. Dérouler le serveur → **IPv4**.
3. Clic droit sur **IPv4 → Nouvelle étendue**.
4. Donner un nom (`LAN10`).
5. Saisir **début** : `192.168.10.100` ; **fin** : `192.168.10.200` ; masque `255.255.255.0`.
6. Si l'énoncé l'exige, déclarer une **plage d'exclusion** à l'intérieur de l'étendue ; sinon passer cette étape.
7. Accepter la durée de bail proposée, sauf autre consigne.
8. Choisir **Configurer ces options maintenant** :
   - **003 Routeur** : `192.168.10.1` si une passerelle existe pour ce réseau ;
   - **006 Serveurs DNS** : `192.168.10.10` ;
   - **015 Nom de domaine DNS** : nom exact du domaine indiqué par le sujet.
9. Terminer et **activer l'étendue**.
10. Dans `IPv4 → Étendue → Pool d'adresses`, contrôler la plage et les éventuelles exclusions.

**Depuis un poste client configuré en DHCP :**

~~~cmd
ipconfig /release
ipconfig /renew
ipconfig /all
~~~

**Résultat attendu :** IP dans la plage, bon masque, bonne passerelle et DNS. Vérifier le **serveur DHCP** affiché par `ipconfig /all`.

**Équivalent PowerShell :**

~~~powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4OptionValue -ScopeId 192.168.10.0
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
~~~

## 9. Cas pratique REA — exclure une plage et réserver une adresse

**Énoncé type :** exclure `.150` à `.159` du pool et réserver `.120` à un poste identifié par son adresse MAC.

### A. Exclusion

1. Dans **DHCP → IPv4 → Étendue → Pool d'adresses**, clic droit → **Nouvelle plage d'exclusion**.
2. Entrer **début** `192.168.10.150`, **fin** `192.168.10.159`.
3. Vérifier que ces adresses ne sont plus attribuables **dynamiquement** par l'étendue.

~~~powershell
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.10.0 -StartRange 192.168.10.150 -EndRange 192.168.10.159
Get-DhcpServerv4ExclusionRange -ScopeId 192.168.10.0
~~~

**Attention :** ne pas réexécuter `Add-` si l'exclusion existe déjà.

### B. Réservation DHCP

1. Sur le poste cible, obtenir l'adresse physique par `ipconfig /all` ; relever la **bonne carte Ethernet/Wi-Fi**.
2. Dans DHCP, ouvrir **Étendue → Réservations → Nouvelle réservation**.
3. Renseigner **Nom**, **Adresse IP** (ex. `192.168.10.120`) et **Adresse MAC** du client.
4. Valider puis vérifier la réservation dans la liste.
5. Sur le client **en DHCP**, exécuter `ipconfig /release` puis `ipconfig /renew`.
6. Contrôler `ipconfig /all` et les **Baux d'adresses**.

**Variante pratique :** si un bail correspondant au bon client existe déjà, clic droit dessus → **Ajouter à la réservation**.

~~~powershell
Get-DhcpServerv4Reservation -ScopeId 192.168.10.0
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
~~~

Une réservation DHCP **ne rend pas statique la carte réseau du client** : la carte doit rester configurée pour obtenir une adresse automatiquement. Choisir une IP autorisée dans l'étendue et non déjà attribuée à une autre machine.

## 10. Cas pratique REA — changer les options 003, 006 et 015

1. Dans **DHCP → IPv4 → Étendue → Options d'étendue**, clic droit → **Configurer les options**.
2. Cocher/configurer :
   - **003 Routeur** : passerelle ;
   - **006 Serveurs DNS** : serveur(s) DNS ;
   - **015 Nom de domaine DNS** : suffixe du domaine.
3. Vérifier les valeurs saisies, puis **OK**.
4. Sur le client, renouveler le bail et vérifier `ipconfig /all`.

**À distinguer :** options du **serveur** (globales) et options de **l'étendue** (propres au sous-réseau). Les options définies au niveau de l'étendue peuvent remplacer celles définies globalement pour les clients concernés.

## 11. Panne REA — le client reçoit une APIPA 169.254.x.x

Une adresse `169.254.x.x` signifie généralement que le client n'a pas obtenu de bail IPv4 DHCP.

**Sur le client :**

~~~cmd
ipconfig /all
ipconfig /release
ipconfig /renew
~~~

**Vérifications dans l'ordre :**

1. L'adaptateur est-il actif et relié au bon réseau ? La carte est-elle en **DHCP automatique** ?
2. L'étendue est-elle **activée** ? Reste-t-il des adresses disponibles ?
3. Le service **Serveur DHCP** est-il démarré ?
4. En environnement AD, le serveur est-il **autorisé** ?
5. Client et serveur sur le même réseau ? Sinon, un **relais DHCP** doit exister sur l'équipement L3 ; le serveur DHCP n'en remplace pas la configuration.
6. Un pare-feu ou une règle réseau bloque-t-il les échanges DHCP ? 

**Sur le serveur :**

~~~powershell
Get-Service DHCPServer
Get-DhcpServerInDC
Get-DhcpServerv4Scope
Get-DhcpServerv4ScopeStatistics -ScopeId 192.168.10.0
Get-DhcpServerv4Lease -ScopeId 192.168.10.0
~~~

**Point essentiel :** un problème de DNS n'explique pas à lui seul l'absence de bail DHCP : distinguer **attribution IP** et **résolution de noms**.

**Preuves :** capture de l'étendue active et de ses options ; réservation ou exclusion ; `ipconfig /all` du client après renouvellement.

**Référence :** [Installer et configurer DHCP — Microsoft Learn](https://learn.microsoft.com/fr-fr/windows-server/networking/technologies/dhcp/quickstart-install-configure-dhcp-server)
