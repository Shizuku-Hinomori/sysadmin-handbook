# Windows — partages SMB, droits NTFS et AGDLP

> Exemple : `D:\Partages\RH` partagé en `RH` ; groupe de domaine `ENTREPRISE\GL_RH_RW`. Adapter aux noms réels.

## 1. Créer le dossier

~~~powershell
New-Item -Path "D:\Partages\RH" -ItemType Directory -Force
~~~

Vérifier que le disque `D:` existe.

## 2. Créer le partage SMB

~~~powershell
New-SmbShare -Name "RH" -Path "D:\Partages\RH" -ChangeAccess "ENTREPRISE\GL_RH_RW"
Get-SmbShare -Name "RH"
Get-SmbShareAccess -Name "RH"
~~~

Remplacer `ENTREPRISE` par le nom NetBIOS réel du domaine. Éviter de supposer que les droits de partage et NTFS sont identiques.

## 3. Donner les droits NTFS (exemple : Modifier)

~~~cmd
icacls "D:\Partages\RH" /grant "ENTREPRISE\GL_RH_RW:(OI)(CI)M"
icacls "D:\Partages\RH"
~~~

- `(OI)` : héritage fichiers ;
- `(CI)` : héritage sous-dossiers ;
- `M` : Modifier ; `RX` : lecture/exécution ; `F` : contrôle total.

Conserver les autorisations nécessaires à `SYSTEM` et aux administrateurs ; ne pas désactiver l'héritage sans consigne.

## 4. Rappel AGDLP

**Utilisateur** → **GG_RH** (groupe global) → **GL_RH_RW** (groupe local de domaine) → **permissions sur le dossier**.

Les droits effectifs combinent les autorisations NTFS et de partage : l'accès réseau ne dépasse pas le plus restrictif des deux.

## 5. Interface graphique

- Dossier → **Propriétés → Partage → Partage avancé** ;
- **Autorisations** du partage ;
- Onglet **Sécurité** : permissions NTFS et groupes.

## 6. Tester depuis un poste client

~~~powershell
Test-Path "\\SRV-FICHIERS\RH"
net use Z: \\SRV-FICHIERS\RH
~~~

Tester avec **un vrai utilisateur membre** de `GG_RH` et vérifier lecture / création / modification selon le besoin.

## Compléments interface graphique

- [Activer l'énumération basée sur l'accès (ABE) et gérer les quotas FSRM](FSRM-Quotas-ABE.md)
- [Profils itinérants et dossiers de base sur un partage SMB](AD-Profils-Scripts.md)


### 📷 Captures correspondant à cette étape

**35 — tester acces partage**

![tester acces partage](../images/ad/ad-35-tester-acces-partage.png)

---

## 7. Cas pratique — créer un partage sécurisé dans l'interface graphique

**Énoncé type :** le dossier `D:\Partages\RH` est accessible en `\\SRV-FICHIERS\RH`. Les membres du service RH doivent pouvoir **lire et modifier**, les autres utilisateurs ordinaires ne doivent pas y accéder.

**Avant de commencer :** vérifier que le disque `D:` existe, ou adapter le chemin à l'énoncé.

1. Dans l'Explorateur sur le **serveur de fichiers**, créer `D:\Partages\RH`.
2. Clic droit sur `RH` → **Propriétés → Partage → Partage avancé**.
3. Cocher **Partager ce dossier**, nommer le partage `RH`.
4. Cliquer sur **Autorisations**. Contrôler les entrées existantes. Donner au groupe **local de domaine `GL_RH_RW`** l'accès de partage approprié (**Modifier** pour l'exemple).
5. Ne pas retirer les accès administrateur/SYSTEM nécessaires. Éviter de laisser un accès en modification à **Tout le monde** si l'énoncé demande un partage réservé.
6. Revenir dans **Propriétés → Sécurité** : onglet des **droits NTFS**.
7. Cliquer sur **Modifier → Ajouter**, sélectionner `GL_RH_RW` et lui attribuer **Modification** ; conserver les droits techniques nécessaires.
8. Fermer les fenêtres, puis vérifier le partage dans `fsmgmt.msc` (**Dossiers partagés**).

**Règle :** pour un accès par le réseau, les permissions **SMB ET NTFS** sont évaluées ; une permission sur le partage ne suffit pas si NTFS refuse l'accès.

## 8. Cas pratique — construire les groupes AGDLP

**Énoncé type :** deux utilisateurs RH doivent accéder au même dossier sans recevoir chacun des permissions NTFS individuelles.

**Dans `dsa.msc` — Utilisateurs et ordinateurs Active Directory :**

1. Dans l'OU adéquate, créer un **groupe de sécurité global** `GG_RH`.
2. Ajouter les comptes RH dans l'onglet **Membres** de `GG_RH`.
3. Créer un **groupe de sécurité de domaine local** `GL_RH_RW`.
4. Ajouter **`GG_RH` en tant que membre de `GL_RH_RW`**.
5. Donner à **`GL_RH_RW`** les droits sur le dossier et le partage, pas directement à chaque utilisateur.

**Chaîne attendue :**

`Utilisateur → GG_RH (global) → GL_RH_RW (domaine local) → droits SMB/NTFS sur RH`

**Vérifier dans PowerShell (sur un poste avec RSAT) :**

~~~powershell
Get-ADGroupMember "GG_RH"
Get-ADGroupMember "GL_RH_RW"
Get-SmbShareAccess -Name "RH"
~~~

Une modification d'appartenance aux groupes peut nécessiter une **nouvelle connexion de l'utilisateur** pour que son jeton de sécurité reflète les nouveaux groupes.

## 9. Cas pratique — tester depuis le bon compte client

**Ne pas tester uniquement avec un compte administrateur**, qui peut disposer d'autres droits.

1. Sur un **poste du domaine**, se connecter avec un utilisateur membre de `GG_RH`.
2. Ouvrir `\\SRV-FICHIERS\RH` depuis l'Explorateur de fichiers.
3. Essayer de **lire**, **créer**, **modifier**, puis **supprimer** un fichier de test, si l'énoncé prévoit ces opérations.
4. Refaire le test avec un utilisateur qui **n'est pas membre** du groupe autorisé.
5. Nettoyer le fichier de test.

**Commandes utiles côté client :**

~~~cmd
whoami
whoami /groups
net use
net use R: \\SRV-FICHIERS\RH
~~~

**Sur le serveur :**

~~~powershell
Get-SmbShare -Name RH
Get-SmbShareAccess -Name RH
Get-Acl "D:\Partages\RH" | Format-List
~~~

**Résultat attendu :** RH accède avec les droits demandés, le compte hors RH ne peut pas modifier ni consulter si aucun autre droit ne le lui permet.

## 10. Dépannage — « Accès refusé » à un partage

| Vérification | Manipulation |
|---|---|
| Le nom du serveur se résout-il ? | `nslookup SRV-FICHIERS` |
| Le partage est-il joignable ? | `Test-NetConnection SRV-FICHIERS -Port 445` dans PowerShell |
| Le partage existe-t-il ? | `Get-SmbShare` sur le serveur |
| Le compte a-t-il le bon groupe ? | `whoami /groups` sur le client, puis vérifier AD |
| Les droits **de partage** autorisent-ils le compte ? | `Get-SmbShareAccess -Name RH` |
| Les droits **NTFS** l'autorisent-ils ? | Propriétés → Sécurité → Avancé → **Accès effectif** |
| Le client utilise-t-il un ancien compte ? | `net use` ; vérifier la session active |

**Attention :** les refus explicites et les droits hérités peuvent influencer l'accès. Ne pas désactiver l'héritage ni supprimer des ACLs pour « essayer ».

## 11. Fichier d'exemple et preuves pour le jury

- Capture du dossier `RH` et de **Partage avancé → Autorisations**.
- Capture de l'onglet **Sécurité** avec `GL_RH_RW`.
- Capture des membres de `GG_RH` et de `GL_RH_RW`.
- Test d'accès depuis un utilisateur du domaine et, si demandé, test de refus depuis un autre.

**Référence :** [Partages et permissions NTFS — Microsoft Learn](https://learn.microsoft.com/fr-fr/iis/web-hosting/configuring-servers-in-the-windows-web-platform/configuring-share-and-ntfs-permissions)
