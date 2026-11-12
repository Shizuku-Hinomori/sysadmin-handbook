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
