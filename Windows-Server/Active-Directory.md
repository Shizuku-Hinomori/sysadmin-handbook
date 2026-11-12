# Active Directory — OU, utilisateurs, groupes et AGDLP

> Exemples : domaine `entreprise.local`, OU `RH`, utilisateur `jdupont`. Adapter les noms et distinguer **DN**, **nom DNS**, **SamAccountName**.

## 1. Vérifier le domaine et le module

~~~powershell
Import-Module ActiveDirectory
Get-ADDomain
Get-ADDomainController -Filter *
Get-ADOrganizationalUnit -Filter *
~~~

Les cmdlets ActiveDirectory sont à exécuter sur un contrôleur de domaine ou un poste avec RSAT et droits appropriés.

## 2. Créer une OU

~~~powershell
$base = (Get-ADDomain).DistinguishedName
New-ADOrganizationalUnit -Name "RH" -Path $base
Get-ADOrganizationalUnit -Filter 'Name -eq "RH"'
~~~

## 3. Créer un utilisateur

~~~powershell
$base = (Get-ADDomain).DistinguishedName
$ou = "OU=RH,$base"
$pwd = Read-Host "Mot de passe initial" -AsSecureString
New-ADUser -Name "Jean Dupont" -GivenName "Jean" -Surname "Dupont" -SamAccountName "jdupont" -UserPrincipalName "jdupont@entreprise.local" -Path $ou -AccountPassword $pwd -Enabled $true -ChangePasswordAtLogon $true
Get-ADUser jdupont -Properties Enabled,DistinguishedName
~~~

Utiliser un suffixe UPN **réellement configuré** si différent du domaine de l'exemple.

### Interface graphique

`Gestionnaire de serveur → Outils → Utilisateurs et ordinateurs Active Directory` → clic droit sur le domaine/OU → **Nouveau** → Unité d'organisation / Utilisateur / Groupe.

## 4. Créer des groupes et ajouter l'utilisateur

~~~powershell
$base = (Get-ADDomain).DistinguishedName
$ou = "OU=RH,$base"
New-ADGroup -Name "GG_RH" -SamAccountName "GG_RH" -GroupScope Global -GroupCategory Security -Path $ou
New-ADGroup -Name "GL_RH_RW" -SamAccountName "GL_RH_RW" -GroupScope DomainLocal -GroupCategory Security -Path $ou
Add-ADGroupMember -Identity "GG_RH" -Members "jdupont"
Add-ADGroupMember -Identity "GL_RH_RW" -Members "GG_RH"
Get-ADGroupMember "GL_RH_RW"
~~~

**AGDLP :** Comptes → Groupe global → Groupe local de domaine → Permissions sur la ressource. Le groupe `GL_RH_RW` doit ensuite recevoir les autorisations NTFS / partage nécessaires.

## 5. Réinitialiser un mot de passe / activer un compte

~~~powershell
$pwd = Read-Host "Nouveau mot de passe" -AsSecureString
Set-ADAccountPassword -Identity jdupont -Reset -NewPassword $pwd
Enable-ADAccount -Identity jdupont
Unlock-ADAccount -Identity jdupont
~~~

## 6. Vérifier la réplication et FSMO si demandé

~~~cmd
repadmin /replsummary
repadmin /showrepl
netdom query fsmo
~~~

**Ne pas transférer les rôles FSMO sans consigne explicite.**

## Parcours complémentaires en interface graphique

- [Installer AD DS et promouvoir un DC](AD-DS-Installation.md)
- [Créer et gérer les comptes et groupes depuis l'interface](AD-Comptes-GUI.md)
- [Profils itinérants, dossiers de base et scripts de connexion](AD-Profils-Scripts.md)
- [Jonction au domaine, gestion des postes, OU et DN](AD-Ordinateurs-OU.md)
- [Checklist des captures personnelles utiles](Captures-a-faire.md)
