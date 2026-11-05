# Chwiya b Chwiya — apprendre le darija

Application web d'apprentissage de l'arabe marocain : cartes de révision,
exercices, conjugaison et suivi de progression. Un seul fichier, sans
framework et sans serveur.

## Mise en ligne sur GitHub Pages

1. Dépôt GitHub > **Settings** > **Pages**
2. *Source* : **Deploy from a branch**, branche `main`, dossier `/ (root)`
3. Attendre une à deux minutes. L'adresse est `https://ziyadakkar.github.io/chwiya-b-chwiya/`

## Synchronisation de la progression (Firebase)

Sans cette configuration, l'application fonctionne normalement : la progression
reste dans le navigateur de l'appareil. Avec, chacun retrouve sa progression sur
tous ses appareils après connexion.

### 1. Créer le projet

1. Aller sur <https://console.firebase.google.com> et **Ajouter un projet**
2. Nom : `chwiya-b-chwiya`. Google Analytics n'est pas nécessaire.

### 2. Activer la connexion par e-mail

**Authentication** > **Get started** > onglet **Sign-in method** >
**E-mail/Mot de passe** > activer le premier interrupteur > **Enregistrer**.

### 3. Autoriser le domaine GitHub Pages

**Authentication** > **Settings** > **Authorized domains** > **Add domain** >
`ziyadakkar.github.io`

### 4. Créer la base

**Firestore Database** > **Créer une base de données** > mode **production** >
région `eur3 (europe-west)`.

Puis onglet **Règles**, remplacer tout par ceci et publier :

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /progression/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

Ces règles garantissent que chacun ne lit et n'écrit que sa propre progression.

### 5. Récupérer les clés

**Paramètres du projet** (roue dentée) > section **Tes applications** >
icône **Web `</>`** > nom `chwiya` > **Enregistrer l'application**.

Firebase affiche un objet `firebaseConfig`. Copier ses six valeurs dans
`firebase-config.js`, puis pousser le fichier.

### 6. Vérifier

Ouvrir l'application, onglet **Progrès**. Le bloc « Sauvegarde en ligne »
apparaît. Créer un compte, puis vérifier dans la console Firebase que le
document est bien créé dans la collection `progression`.

## Fonctionnement de la synchronisation

- La progression est écrite dans `localStorage` à chaque réponse : l'application
  reste utilisable hors connexion.
- Une fois connectée, elle est envoyée vers Firestore toutes les cinq secondes
  si elle a changé, et quand l'onglet passe en arrière-plan.
- À la connexion, les données locales et distantes sont **fusionnées** en gardant
  la meilleure valeur des deux côtés. Rien n'est perdu en changeant d'appareil.
- Les réglages (thèmes sélectionnés, écriture arabe, objectif) restent propres à
  l'appareil courant.

## Structure

| Fichier | Rôle |
|---|---|
| `index.html` | Toute l'application : HTML, CSS, JavaScript, contenu pédagogique |
| `firebase-config.js` | Les clés Firebase. Seul fichier à modifier pour la synchronisation |

Le contenu pédagogique se trouve en haut du `<script>` d'`index.html` :
`DATA` pour le vocabulaire, `VERBS` pour la conjugaison, `SENTENCES` pour
l'exercice de construction de phrases.

## Conventions

Le darija s'écrit en lettres latines avec les chiffres usuels :
`3` = ع, `7` = ح, `q` = ق, `kh` = خ, `gh` = غ, `2` = ء.
L'écriture arabe est toujours fournie en parallèle.
