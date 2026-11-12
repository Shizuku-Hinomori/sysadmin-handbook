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
