# Captures utiles — Active Directory et Windows Server

> **Les 48 captures du document de formation sont répertoriées dans la [galerie complète](../images/ad/README.md).** Elles sont intégrées aux fiches de manipulation après importation des PNG dans le dossier `images/ad/`.

> Cette page sert de **checklist** pour réaliser des captures depuis **ton propre labo**.  Si le dépôt devient public, ne publier aucune donnée confidentielle.

## Priorité 1 — manipulations probables

- [ ] **Installation AD DS** : rôle sélectionné, option nouvelle forêt / promotion et écran de vérification.
- [ ] **Création d'utilisateur** : nouvelle OU, assistant utilisateur, propriétés `Compte`.
- [ ] **Groupes** : création d'un groupe **global** et **local de domaine**, onglets `Membres` / `Membre de`.
- [ ] **Jonction domaine** : DNS du client, fenêtre de jonction et confirmation.
- [ ] **Partage NTFS** : fenêtre de droits **Partage** et fenêtre de droits **Sécurité**.

## Priorité 2 — manipulations spécifiques

- [ ] **Profil itinérant** : propriétés AD → `Profil` → chemin UNC.
- [ ] **Dossier de base** : `Profil` → lecteur + chemin personnel.
- [ ] **Script de connexion** : contenu du `.bat`, stockage dans NETLOGON et champ du profil.
- [ ] **ABE** : propriétés du partage avec la case cochée.
- [ ] **Quota FSRM** : assistant quota et taille/type sélectionnés.
- [ ] **OU / DN** : arborescence et `Éditeur d'attributs → distinguishedName`.
- [ ] **Compte AD** : horaires de connexion, date d'expiration et changement de mot de passe.

## Conseils pour de bonnes captures

1. Ne garder que la fenêtre utile, avec la case ou le champ visé bien visible.
2. Nommer clairement les fichiers, par exemple `ad-creer-utilisateur.png`.
3. Masquer **mots de passe, clés, identifiants privés, adresses publiques ou informations d'entreprise**.
4. Lorsque les images sont prêtes et autorisées à être publiées, les ajouter dans un dossier `images/ad/` puis créer les liens dans les fiches correspondantes.

**Rappel :** pour l'examen, les procédures textuelles ci-dessus restent utilisables même sans images.

## Captures sélectionnées pour intégration

Une sélection de **7 captures originales** a été préparée pour les fiches ci-dessous. Leur import dans `images/ad/` est nécessaire pour afficher les illustrations :

- `ad-installation-nouvelle-foret.png` → installation AD DS
- `ad-creation-utilisateur.png` et `ad-creation-groupe.png` → comptes et groupes
- `ad-profil-itinerant.png` → profils
- `ad-jonction-domaine.png` → ordinateurs
- `ad-enumeration-abe.png` et `ad-quota-fsrm.png` → partages, ABE et quotas
