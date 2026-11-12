# Hyper-V — cluster, états et déplacements de VM

## Consulter le cluster

Depuis un nœud avec le module FailoverClusters :

~~~powershell
Get-Cluster
Get-ClusterNode
Get-ClusterGroup
Get-ClusterResource
~~~

Pour lister les VM sur un hôte Hyper-V :

~~~powershell
Get-VM
Get-VM | Select-Object Name, State, ComputerName
~~~

## Déplacer une VM clusterisée (interface graphique)

1. Ouvrir **Gestionnaire du cluster de basculement**.
2. Aller dans **Rôles**.
3. Sélectionner la VM concernée.
4. **Déplacer** → choisir la méthode proposée → sélectionner le nœud.
5. Vérifier le **nouveau propriétaire** et l'état de la VM.

## Différences

- **Migration dynamique (Live Migration)** : la VM continue de fonctionner pendant le transfert ; mémoire et état d'exécution sont migrés. Dans un cluster à stockage partagé, les disques de VM peuvent rester en place.
- **Migration rapide (Quick Migration)** : mise en état enregistré/sauvegardé puis reprise sur le nœud cible ; **interruption de service**, généralement plus longue qu'une migration dynamique.
- **Basculement automatique** : suite à la défaillance d'un nœud, le cluster tente de redémarrer les rôles sur un nœud disponible si les dépendances et le quorum le permettent.

**Attention :** la migration rapide ne signifie pas forcément « transfert de tous les fichiers de disque ». Les détails dépendent du stockage.

## Avant / après

- Vérifier l'état des nœuds, quorum, réseau de cluster et accès au stockage partagé.
- Ne pas éteindre ou débrancher un nœud sans instruction.
- Vérifier le propriétaire du rôle VM après la migration et sa disponibilité dans l'invité.

## Si un rôle est « non contrôlé » / « inconnu »

Consulter son état exact, la connexion entre les nœuds, le quorum, les services de cluster et la disponibilité du stockage. Le rôle ne bascule pas nécessairement simplement parce que deux câbles ont été retirés : le résultat dépend de la topologie et des règles du cluster.
