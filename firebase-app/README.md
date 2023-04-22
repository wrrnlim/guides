# Creating a Firebase App

Example Firebase App following this [tutorial](https://www.youtube.com/watch?v=q5J5ho7YUhA). Some instructions detailed below are different from the video as we are using Firebase 9. Run this demo with `npm start`.

## Setting Up

View this [repo](https://github.com/wrrnlim/firebase-connect) for examples.

1. Start a new project in Firebase, then select ⚙ -> project settings -> and choose your app type (Android, iOS, web) to add a new app
2. Install Firebase in your app via `npm install firebase` in your project terminal
3. Copy the SDK that Firebase gives you. Here it is added in [main.js](https://github.com/wrrnlim/firebase-connect/blob/main/public/main.js).
4. We then need to import the components we need in our app. These are in the form of `import { <component> } from 'firebase/<service>'`. You can view available SDKs on the [docs](https://firebase.google.com/docs/web/setup#available-libraries).
5. Module imports do not work in browsers without module bundlers. A workaround is to use browser modules by importing from CDN links, as displayed in [modules.js](https://github.com/wrrnlim/firebase-connect/blob/main/public/modules.js). These are in the form of `import { <component> } from 'https://www.gstatic.com/firebasejs/9.0.0/firebase-<service>.js'`. Remember to add `type="module"` in the script tag in HTML. For production, module bundlers should be used.
6. If using browser modules, we can serve a local server with `npx serve <directory>` and view our site there.

## Connecting to Firebase with Firebase CLI

After setting up Firebase connection, we can use the Firebase CLI. If you do not have it installed, you can install it with `npm install firebase-tools -g`. Make sure you have node `>=v16.2`. Once that is installed, login to Firebase with

```bash
firebase login
```

then

```bash
firebase init
```

and choose the services you wish to include to initialize the project. For this project, hosting and emulator is included. `npm start` serves the app (via hosting), and runs the `firebase emulators:start` command.

## User Authentication

Navigate to the [Firebase Console](https://console.firebase.google.com/) and choose "Authentication." Here, you can add sign-in methods such as email/password login or Google login. The docs to authenticate via Google can be found [here](https://firebase.google.com/docs/auth/web/google-signin), or you may follow below. To authenticate a user, we first need the following modules from `firebase/auth`: `getAuth`, `onAuthStateChanged`, `signInWithPopup`, `GoogleAuthProvider`, `signOut`. In [`app.js`](public/app.js),

```js
import { getAuth, onAuthStateChanged, signInWithPopup, GoogleAuthProvider, signOut } from 'https://www.gstatic.com/firebasejs/9.0.0/firebase-auth.js' // if using browser modules

// OR

import { getAuth, onAuthStateChanged, signInWithPopup, GoogleAuthProvider, signOut } from 'firebase/auth'
```

Construct the auth with

```js
const auth = getAuth(app);
```

We can then set an on click listener on a login button that would call

```js
signInWithPopup(auth, provider)
  .then(result => {
    console.log(result.user);
  });
```

A log out button would call

```js
signOut(auth);
```

We can keep track of login state with

```js
onAuthStateChanged(auth, user => {
  if (user) {
    // show log out button; change UI accordingly
  } else {
    // show log in button; change UI accordingly
  }
});
```

`user.displayName` will get the Google account's name, and `user.uid` will show the user's ID unique to your app. You can view registered users and their IDs in Firebase Console.

## Cloud Firestore

### Connect with Realtime Updates

In your project's [Firebase Console](https://console.firebase.google.com/), navigate to "Firestore Database" and create a new collection. For now, we can leave it in test mode and set security rules later. The Firestore docs can be found [here](https://firebase.google.com/docs/firestore/manage-data/add-data), or follow the steps below. To connect your project to Firestore, we need the following modules from `firebase/firestore` in [`app.js`](public/app.js).

```js
import { getFirestore, serverTimestamp, addDoc, collection, query, where, onSnapshot } from 'https://www.gstatic.com/firebasejs/9.0.0/firebase-firestore.js' // browser module

//OR

import { getFirestore, serverTimestamp, addDoc, collection, query, where, onSnapshot } from firebase/firestore
```

Get a reference to Firestore with

```js
const db = getFirestore(app)
```

We can get a reference to your collection using

```js
const collectionRef = collection(db, 'collectionName');
```

To add a document to the collection:

```js
const docRef = addDoc(collectionRef, {
  name: 'banana'
  color: 'yellow'
  createdAt: serverTimestamp()
});
```

To listen to [realtime updates](https://firebase.google.com/docs/firestore/query-data/listen) for multiple documents, we can use a query to find documents we want to listen to:

```js
const q = query(collectionRef, where('uid', '==', user.uid)); // set a query
const unsubscribe = onSnapshot(q, (querySnapshot) => {
  const items  = []; // list of items we want
  querySnapshot.forEach((doc) => {
    items.push(`<li>${doc.data().name}</li>`); // store as list items so we can place into a <ul></ul>
  });
  itemList.innerHTML = items.join('');  // itemList is a <ul></ul> element
});
```

To stop listenting to changes, we can simply use

```js
unsubscribe()
```

### Security Rules

From the Firestore Console, navigate to the "Rules" tab. Here we can write the following security rules.

Firstly, we want to disable all read and writes. Since we selected test mode at the beginning, Firestore is currently allowing all reads and writes if the request time is within 30 days of creation. We want to change this to not allow any read/write.

```js
match /{document=**} {
  allow read, write: if false;
}
```

The we want to add a new security rule to allow users to read if the resource contains their user ID, and to write if what they are writing also contains their user ID.

```js
match /myCollection/{docId} {
  allow write: if request.auth.uid == request.resource.data.uid;
  allow read: if request.auth.uid == resource.data.uid;
}
```

Remember to replace `myCollection` with your collection name.

The final security rules should look like this:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
    match /myCollection/{docId} {
      allow write: if request.auth.uid == request.resource.data.uid;
      allow read: if request.auth.uid == resource.data.uid;
    }
  }
}
```

## Deploying to Hosting

To deploy your site to the web, run

```bash
firebase deploy
```

To unhost, run

```bash
firebase hosting:disable
```

If you want to delete the deployment, go to Firebase console -> hosting, and you can delete the deploy.
