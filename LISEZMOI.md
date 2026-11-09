# Entre Nous : mettre l'application en ligne et l'installer

Cette application est un salon de discussion où chacun écrit dans sa langue et lit les messages des autres traduits dans la sienne.

Pour qu'elle fonctionne à plusieurs, il faut deux services gratuits :
- **Firebase** (Google) garde les messages et identifie chaque téléphone.
- **Netlify** (ou un autre hébergeur) met l'application en ligne à une adresse web.

Comptez 20 à 30 minutes, sur un ordinateur. Les noms des boutons peuvent changer légèrement d'un moment à l'autre, mais la démarche reste la même.

---

## Étape 1 : créer le projet Firebase

1. Allez sur https://console.firebase.google.com et connectez-vous avec un compte Google.
2. Cliquez sur **Créer un projet**, donnez-lui un nom (par exemple `entre-nous`). Vous pouvez refuser Google Analytics.

## Étape 2 : relier l'application au projet

1. Dans le projet, cliquez sur l'icône **Web** (`</>`) pour ajouter une application web. Donnez-lui un nom, sans cocher « Firebase Hosting ».
2. Firebase affiche un bloc de code avec `apiKey`, `authDomain`, `projectId`, etc.
3. Ouvrez le fichier **config.js** (avec le Bloc-notes, par exemple) et remplacez chaque `COLLER_ICI` par la valeur correspondante. Gardez les guillemets. Enregistrez.

## Étape 3 : activer la connexion anonyme

1. Dans Firebase, ouvrez **Authentication**, puis **Commencer**.
2. Dans l'onglet **Sign-in method**, activez **Anonyme** et enregistrez.

Chaque téléphone reçoit ainsi un identifiant automatique, sans mot de passe.

## Étape 4 : créer la base de messages

1. Ouvrez **Firestore Database**, puis **Créer une base de données**.
2. Choisissez le mode **production** et un emplacement en Europe.
3. Ouvrez l'onglet **Règles**, remplacez tout le contenu par le texte ci-dessous, puis cliquez sur **Publier**.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /members/{uid} {
      allow read: if request.auth != null;
      allow write: if request.auth != null
        && request.auth.uid == uid
        && request.resource.data.name.size() <= 24;
    }
    match /messages/{id} {
      allow read: if request.auth != null;
      allow create: if request.auth != null
        && request.resource.data.uid == request.auth.uid
        && request.resource.data.text.size() <= 2000;
      allow update, delete: if request.auth != null
        && resource.data.uid == request.auth.uid;
    }
  }
}
```

Ces règles permettent à chacun de lire le salon, d'écrire sous son propre nom, et de supprimer uniquement ses propres messages.

## Étape 5 : mettre l'application en ligne

Avec Netlify :

1. Décompressez l'archive `entre-nous-app.zip` pour obtenir le dossier `entre-nous-app` (avec `config.js` déjà modifié).
2. Allez sur https://app.netlify.com/drop et créez un compte gratuit si on vous le demande.
3. Glissez le **dossier** `entre-nous-app` dans la zone indiquée.
4. Netlify vous donne une adresse du type `https://nom-au-hasard.netlify.app`. Vous pouvez la renommer dans les réglages du site.

Si l'application affiche une erreur d'autorisation à l'ouverture, retournez dans Firebase, **Authentication**, **Paramètres**, **Domaines autorisés**, et ajoutez votre adresse Netlify.

Si vous modifiez `config.js` plus tard, glissez de nouveau le dossier dans Netlify pour mettre en ligne la nouvelle version.

## Étape 6 : installer sur le téléphone

Ouvrez l'adresse de l'application sur le téléphone.

- **Android (Chrome)** : ouvrez le menu (trois points) et choisissez **Installer l'application** ou **Ajouter à l'écran d'accueil**.
- **iPhone (Safari)** : touchez le bouton **Partager**, puis **Sur l'écran d'accueil**.

L'icône Entre Nous apparaît avec les autres applications et s'ouvre en plein écran.

## Inviter du monde

Envoyez simplement l'adresse de l'application. Chaque personne choisit son nom et sa langue à la première ouverture. Toute personne qui a le lien peut rejoindre le salon, donc ne le partagez qu'avec ceux que vous voulez inviter.

---

## Bon à savoir

- **Traduction.** Elle utilise le service gratuit MyMemory, sans clé. La limite quotidienne est d'environ 5 000 caractères par connexion internet, et plus haut si vous renseignez une adresse e-mail dans `contactEmail` de `config.js`. Ces limites peuvent changer. Quand elle est atteinte, un message s'affiche et les messages restent lisibles dans leur langue d'origine. La qualité est correcte pour des phrases courantes, moins fine que celle d'une traduction faite par Claude.
- **Aperçu.** Tant que `config.js` contient `COLLER_ICI`, l'application s'ouvre en mode aperçu avec des messages d'exemple, et rien n'est partagé.
- **Téléphone perdu ou données effacées.** Comme il n'y a pas de mot de passe, un téléphone dont les données du navigateur sont effacées devient un nouveau participant. Ses anciens messages restent dans le salon mais il ne peut plus les supprimer.
- **Gratuité.** Les offres gratuites de Firebase et de Netlify suffisent largement pour un petit groupe.
