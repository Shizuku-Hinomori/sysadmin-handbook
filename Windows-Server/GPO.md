# GPO — créer, lier et vérifier une stratégie

## 1. Interface graphique (méthode rapide)

1. `Gestionnaire de serveur → Outils → Gestion des stratégies de groupe` (`gpmc.msc`).
2. Développer **Forêt → Domaines → domaine → OU cible**.
3. Clic droit sur l'OU → **Créer un objet GPO dans ce domaine, et le lier ici**.
4. Donner un nom explicite, puis clic droit → **Modifier**.
5. Paramétrer le bon volet : **Configuration ordinateur** ou **Configuration utilisateur**.
6. Vérifier le **lien**, l'OU des objets ciblés et le **filtrage de sécurité**.

**Rappel :** les OU contiennent/organisent les objets et reçoivent les liens GPO ; les groupes servent notamment à gérer des droits et filtrer certaines applications de stratégie.

## 2. PowerShell (sur serveur avec module GroupPolicy)

~~~powershell
Import-Module GroupPolicy
$base = (Get-ADDomain).DistinguishedName
New-GPO -Name "GPO_RH"
New-GPLink -Name "GPO_RH" -Target "OU=RH,$base" -LinkEnabled Yes
Get-GPO -Name "GPO_RH"
Get-GPInheritance -Target "OU=RH,$base"
~~~

La création et le lien ne définissent **aucun paramètre** : modifier la GPO ensuite via GPMC ou des commandes adaptées à la stratégie demandée.

## 3. Forcer et vérifier sur un poste client

~~~cmd
gpupdate /force
gpresult /r
gpresult /h C:\Temp\rapport-gpo.html
~~~

Créer `C:\Temp` avant d'y écrire si besoin. Pour voir les paramètres ordinateur, ouvrir une invite avec droits suffisants. Certaines modifications nécessitent déconnexion/reconnexion ou redémarrage.

## 4. Diagnostic rapide

Vérifier que l'objet ordinateur ou utilisateur est dans la **bonne OU**, que la GPO est liée et activée, que les autorisations/filtrages conviennent et que le poste accède au domaine.

---

## 5. Cas pratique REA — interdire le Panneau de configuration à une OU

**Énoncé type :** les utilisateurs de l'OU `Stagiaires` ne doivent plus accéder au Panneau de configuration ni aux Paramètres Windows. Les autres utilisateurs ne doivent pas être concernés.

**Sur le contrôleur de domaine ou un serveur avec GPMC :**

1. Vérifier dans `dsa.msc` que les **comptes utilisateurs** à cibler sont bien dans l'OU `Stagiaires`. Ne pas confondre OU et groupe.
2. Ouvrir `gpmc.msc` : **Gestion des stratégies de groupe**.
3. Dérouler **Forêt → Domaines → votre domaine → OU Stagiaires**.
4. Clic droit sur l'OU → **Créer un objet GPO dans ce domaine, et le lier ici** ; nommer `GPO_Blocage_Panneau`.
5. Clic droit sur la GPO → **Modifier**.
6. Aller dans **Configuration utilisateur → Stratégies → Modèles d'administration → Panneau de configuration**.
7. Ouvrir la stratégie **Interdire l'accès au Panneau de configuration et à l'application Paramètres du PC** (libellé susceptible de varier) et choisir **Activé**.
8. Fermer l'éditeur. Dans GPMC, contrôler que la GPO est **liée à la bonne OU** et que le lien est activé.

**Sur le poste client, avec une session ouverte en tant qu'utilisateur de l'OU :**

~~~cmd
whoami
gpupdate /force
gpresult /r
~~~

Déconnecter/reconnecter la session si le changement n'est pas visible, puis tenter d'ouvrir le Panneau de configuration.

**Résultat attendu :** accès refusé pour un utilisateur ciblé ; un utilisateur hors périmètre ne doit pas être impacté.

**Si échec :** vérifier l'OU contenant **l'utilisateur**, le lien GPO, le filtrage de sécurité, la résolution du domaine et le compte réellement connecté. Pour obtenir un rapport détaillé, voir le diagnostic plus bas.

## 6. Cas pratique REA — mapper un lecteur réseau par GPO

**Énoncé type :** à l'ouverture de session, les utilisateurs de l'OU `RH` doivent recevoir le lecteur `R:` pointant sur `\\SRV-FICHIERS\RH`.

**Avant la GPO :** vérifier que le partage existe et que **l'utilisateur cible a les droits SMB et NTFS**. Une GPO ne corrige pas un partage inaccessible.

1. Dans `gpmc.msc`, créer `GPO_Lecteur_RH`, **liée à l'OU contenant les utilisateurs RH**.
2. Modifier la GPO, puis aller dans **Configuration utilisateur → Préférences → Paramètres Windows → Mappages de lecteurs**.
3. Clic droit dans la partie droite → **Nouveau → Lecteur mappé**.
4. Dans **Général** :
   - **Action : Mettre à jour** (`Update`).
   - **Emplacement :** `\\SRV-FICHIERS\RH` (à adapter).
   - **Lettre de lecteur :** `R:`.
   - **Reconnecter :** selon l'énoncé.
5. Valider. Ne pas choisir **Supprimer**, qui effacerait le mappage.
6. Vérifier le lien GPO et, si un filtrage est demandé, les utilisateurs/groupes ciblés.
7. Sur le **poste client connecté avec un compte RH**, lancer :

~~~cmd
gpupdate /force
gpresult /r
net use
~~~

Si le lecteur n'apparaît pas, **fermer puis rouvrir la session** et vérifier l'Explorateur de fichiers.

**Résultat attendu :** `R:` apparaît et l'utilisateur peut ouvrir `\\SRV-FICHIERS\RH` avec ses droits.

**Important :** `Configuration utilisateur` agit sur le **compte connecté** ; l'OU de l'ordinateur ne suffit pas pour ce scénario. Une préférence de mappage et ses options d'onglet **Commun** (ciblage/suppression) peuvent modifier le comportement.

## 7. Cas pratique REA — bloquer l'accès au Gestionnaire des tâches

**Énoncé type :** empêcher les utilisateurs d'une OU de lancer le Gestionnaire des tâches.

1. Lier une GPO à l'OU des **utilisateurs** concernés.
2. Ouvrir **Configuration utilisateur → Stratégies → Modèles d'administration → Système → Options Ctrl+Alt+Suppr**.
3. Activer **Supprimer le Gestionnaire des tâches**.
4. Sur le client : `gpupdate /force`, puis reconnexion si nécessaire.
5. Tester `Ctrl + Maj + Échap` avec un compte ciblé.

**Résultat attendu :** le Gestionnaire des tâches est bloqué pour cet utilisateur.

## 8. Vérifier une GPO et comprendre pourquoi elle ne s'applique pas

**Sur le client concerné, dans une invite de commandes :**

~~~cmd
whoami
ipconfig /all
nslookup entreprise.local
gpupdate /force
gpresult /r
gpresult /h "%TEMP%\rapport-gpo.html"
~~~

Le fichier HTML se consulte dans un navigateur. Vérifier les rubriques **GPO appliquées** et **GPO refusées**.

| Symptôme | Premier contrôle |
|---|---|
| La GPO n'apparaît pas dans `gpresult` | Bonne OU ? lien activé ? bon utilisateur ? filtrage ? |
| La GPO apparaît, mais rien ne change | Mauvais volet utilisateur/ordinateur ? session à rouvrir ? mauvais paramètre ? |
| Lecteur mappé absent | Le partage UNC fonctionne-t-il directement ? préférences de lecteurs ? |
| `gpupdate` échoue | DNS du domaine, accès au DC et journaux |
| Seul un poste échoue | Tester sur un autre poste puis examiner les journaux locaux |

**Journaux :** `eventvwr.msc` → **Journaux des applications et services → Microsoft → Windows → GroupPolicy → Operational**.

Ne pas utiliser un compte administrateur de domaine pour valider une restriction visant un utilisateur ordinaire : ouvrir une session **avec le compte réellement ciblé**.

## 9. Preuves faciles à montrer au jury

- Dans GPMC : **nom de la GPO**, **OU liée**, **paramètre Activé**.
- Sur le client : résultat `gpresult /r` ou rapport HTML.
- Résultat concret : restriction visible ou lecteur `R:` présent.
- Tester aussi un compte hors périmètre si l'énoncé exige une application ciblée.

**Références :** [Préférences GPO — Microsoft Learn](https://learn.microsoft.com/fr-fr/windows-server/identity/ad-ds/manage/group-policy/group-policy-preferences) · [Dépannage application GPO](https://learn.microsoft.com/fr-fr/troubleshoot/windows-server/group-policy/applying-group-policy-troubleshooting-guidance)
