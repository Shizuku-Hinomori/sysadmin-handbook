# Sysadmin Handbook

**Procédures pratiques pour l'administration systèmes, réseaux et infrastructures IT.**

Documentation en français, avec commandes à adapter à l'environnement. Pensé pour retrouver rapidement **une manipulation complète** : objectif → étapes → commandes → vérifications.

> ⚠️ **Avant chaque exécution :** remplacer les IP, domaines, utilisateurs, noms de serveurs et chemins d'exemple. Vérifier l'énoncé et les droits. Les commandes ne sont pas garanties adaptées à toutes les versions/configurations ; tester sur un environnement prévu à cet effet. Ne pas publier de mots de passe, clés privées ni configurations confidentielles.

## 🧰 Manipulations pratiques — à ouvrir pendant les manipulations

| Situation | Guide |
|---|---|
| **Créer une GPO, bloquer une fonction, mapper un lecteur, vérifier l'application** | [GPO — cas pratiques](Windows-Server/GPO.md#5-cas-pratique--interdire-le-panneau-de-configuration-à-une-ou) |
| **Créer une étendue DHCP, réservation, exclusion, APIPA** | [DHCP — cas pratiques](Windows-Server/DHCP.md#8-cas-pratique--créer-une-étendue-dhcp-entièrement-dans-la-console) |
| **Créer un partage RH, droits NTFS/SMB, groupes AGDLP, test client** | [Partages SMB — cas pratiques](Windows-Server/Partages-SMB.md#7-cas-pratique--créer-un-partage-sécurisé-dans-linterface-graphique) |
| **Résoudre une panne Windows ou Debian et prouver le résultat** | [Diagnostic de pannes — Windows et Debian](Windows-Server/Diagnostic-Pannes.md) |

**Approche pratique :** énoncé type → manipulation GUI → commandes ciblées → contrôle du résultat. Ne pas exécuter toutes les commandes sans vérifier le contexte.

## 🐧 Linux — Debian

| Manipulation | Fiche |
|---|---|
| Créer un compte, se connecter par clé, sécuriser le SSH | [SSH](Linux/SSH.md) |
| Installer Apache, créer un VirtualHost et publier un site | [Apache](Linux/Apache.md) |
| Créer utilisateurs, groupes et homes | [Utilisateurs et groupes](Linux/Utilisateurs-Groupes.md) |
| Modifier droits, propriétaires, fichiers et dossiers | [Permissions](Linux/Permissions.md) |
| Configurer IP, route, DNS et résolution | [Réseau et DNS](Linux/Reseau-DNS.md) |
| Gérer services, logs et paquets APT | [Services](Linux/Services.md) |

## 🪟 Windows Server

| Manipulation | Fiche |
|---|---|
| Créer OU, utilisateurs, groupes, AGDLP (PowerShell) | [Active Directory](Windows-Server/Active-Directory.md) |
| **Installer AD DS / promouvoir un contrôleur de domaine** | [Installation AD DS](Windows-Server/AD-DS-Installation.md) |
| **Créer et gérer comptes/groupes dans l'interface** | [AD – comptes et groupes (GUI)](Windows-Server/AD-Comptes-GUI.md) |
| **Profils itinérants, dossiers de base, scripts de connexion** | [Profils et scripts](Windows-Server/AD-Profils-Scripts.md) |
| **Joindre un ordinateur, OU, attributs et DN** | [Ordinateurs et OU](Windows-Server/AD-Ordinateurs-OU.md) |
| **ABE et quotas FSRM** | [ABE et FSRM](Windows-Server/FSRM-Quotas-ABE.md) |
| **Captures à réaliser sur son propre labo** | [Checklist captures AD](Windows-Server/Captures-a-faire.md) |
| Créer, lier et appliquer une GPO | [GPO](Windows-Server/GPO.md) |
| Installer DHCP, étendue, options et réservation | [DHCP](Windows-Server/DHCP.md) |
| Créer des zones et enregistrements DNS | [DNS](Windows-Server/DNS.md) |
| Créer partage SMB et attribuer droits NTFS | [Partages SMB](Windows-Server/Partages-SMB.md) |
| Commandes essentielles en PowerShell | [PowerShell](Windows-Server/PowerShell.md) |

**Visuels Windows :** les 48 captures sont maintenant visibles directement dans les fiches correspondantes, à côté des manipulations.

### 📷 Captures Active Directory

[**Galerie complète des 48 captures AD et Windows Server**](images/ad/README.md) — classées par procédure, accessibles depuis chaque fiche, avec les manipulations simples également illustrées.

## 🖥️ Virtualisation

| Manipulation | Fiche |
|---|---|
| Vérifier un cluster et déplacer une VM Hyper-V | [Hyper-V](Virtualisation/Hyper-V.md) |

---

## Méthode d'utilisation

1. Ouvrir directement la fiche depuis le tableau.
2. Lire les **pré-requis** et remplacer les valeurs d'exemple.
3. Exécuter les étapes dans l'ordre.
4. Faire le **test final** et conserver les preuves/captures demandées.

Les captures ont été fournies par l'auteur des manipulations ; avant de rendre le dépôt public, contrôler les noms, identifiants et adresses qui apparaissent à l'écran.

La documentation peut être enrichie après la formation : elle n'est pas liée à un titre professionnel particulier.
