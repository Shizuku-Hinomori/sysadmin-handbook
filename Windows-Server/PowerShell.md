# PowerShell — commandes rapides pour la mise en situation

## Aide et découverte

~~~powershell
Get-Help New-ADUser -Examples
Get-Command *Dhcp*
Get-Command -Module ActiveDirectory
Get-Module -ListAvailable
~~~

## Services et réseau

~~~powershell
Get-Service
Get-Service -Name DNS,DHCPServer
Get-NetIPAddress
Get-NetIPConfiguration
Get-DnsClientServerAddress
Test-Connection 192.168.10.10 -Count 2
Test-NetConnection 192.168.10.10 -Port 445
Resolve-DnsName web.lab.example
~~~

Adapter les noms des services installés.

## IP statique sur interface sélectionnée (exemple seulement)

~~~powershell
Get-NetAdapter
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.10.20 -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
~~~

**Attention :** vérifier d'abord l'interface, l'IP existante et la passerelle. Une modification peut couper la connexion distante. Ne pas ajouter une seconde passerelle par défaut sans besoin.

## Fichiers, dossiers et vérifications

~~~powershell
Get-ChildItem C:\
New-Item -Path "C:\Temp" -ItemType Directory -Force
Get-Content "C:\Temp\test.txt"
Get-EventLog -LogName System -Newest 20
Get-WinEvent -LogName System -MaxEvents 20
~~~

## Commandes classiques à retenir

~~~cmd
ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /flushdns
gpupdate /force
gpresult /r
whoami /groups
net share
~~~

## Scripts PowerShell

~~~powershell
Get-ExecutionPolicy -List
.\script.ps1
~~~

Lire le script avant exécution ; vérifier son dossier courant, ses chemins et dépendances. Ne pas désactiver globalement les protections par automatisme.
