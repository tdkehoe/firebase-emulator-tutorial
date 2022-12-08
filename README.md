# Writing Firebase Cloud Functions on the Firebase Emulator

This tutorial will show you how to write one simple Firebase Cloud Function that reads and writes to Firestore using the Firebase Local Emulator Suite.

The [official documentation](https://firebase.google.com/docs/emulator-suite) is good. This tutorial covers items left out of the official documentation.

## Why Use the Firebase Local Emulator Suite?

You don't have to use the Emulator. You could deploy your Cloud Functions to the Firebase Cloud, trigger them and see the results logged in your Firebase Console. Deploying a function takes a few minutes. You should then wait a minute for your code to propagate. Waiting for the results to log in the console takes a few minutes. Looking at my logs, my cycle time for writing code, deploying, running the function, looking at the results, fixing my code, and deploying the updated code is ten minutes. With emulator this cycle time is reduced to one minute. That's a 10x improvement in development time!

Another reason to use the emulator is that infinite loops won't cost you anything. If you mistakenly deploy an infinite loop to the cloud, Google will bill you for the computing time. And there's no way to kill a function other than deploying updated code. Your infinite loop will likely run for ten minutes.

## Which Emulator(s) To Use?

The Firebase Local Emulator Suite consists of seven emulators: Auth, Realtime Database, Firestore, Storage, and, in beta, Functions, Pub/Sub, and Extentions. This tutorial will use Functions, Firestore, and Storage.

The official documentation mentions several [other tools for prototypes and testing](https://firebase.google.com/docs/emulator-suite#other_tools_for_prototyping_and_testing). I haven't tried these, they might work better for some stuff.

## Install and initialize Firebase

Make a new directory.

```
mkdir EmulatorTutorial
cd EmulatorTutorial
```

Follow the [Getting Started](https://firebase.google.com/docs/emulator-suite/connect_and_prototype) instructions.

Install the Firebase CLI tools.

```
npm install firebase-tools
```

Initialize Firebase, including the emulators.

```
firebase init
```

## Choose a Firebase project

```
firebase use
```

## Connect your functions to Firebase

Find your credentials in the Firebase console.

```
const firebaseConfig = {
  apiKey: "...",
  authDomain: "my-app.firebaseapp.com",
  // credential: admin.credential.cert(serviceAccount),
  databaseURL: "https://my-app.firebaseio.com",
  projectId: "my-app",
  storageBucket: "my-app.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const db = getFirestore(app);
connectFirestoreEmulator(db, 'localhost', 8080);

## Start Emulator

Gentlemen, start your emulator!

```
firebase emulators:start
```

In a minute you should see

```
 ✔  All emulators ready! It is now safe to connect your app. │
│ i  View Emulator UI at http://127.0.0.1:4000/   
```

Open your browser to `http://127.0.0.1:4000/ `. You should see the Firebase Emulator Suite. 

## Write your functions

Open `functions/index.js`.  We'll make a function that converts a message to UPPERCASE.

```js
import * as functions from "firebase-functions";
import { getFirestore, connectFirestoreEmulator } from "firebase/firestore";

export const MakeUppercase = functions.firestore.document('Messages/{docId}').onCreate((snap, context) => {
  try {
    const original = snap.data().original;
    // console.log(context.params.docId);
    functions.logger.log('Uppercasing', context.params.docId, original);
    const uppercase = original.toUpperCase();
    return snap.ref.set({ uppercase }, { merge: true }); // writes to the same document
    // return admin.firestore().collection('AnotherCollection').doc(context.params.docId).set({ uppercase }, { merge: true }); // writes to a different collection
  } catch (error) {
    console.error(error); // emulator always throws an "unhandled error": "Your function timed out after ~60s."
  }
});
```

This is a little different from the example code in the documentation. They set up `index.js` as a CommonJS module, I set up an ES module. I added a line to log the `docId`. I also added a line to write to a different collection. You can leave these lines commented out.

## Trigger your cloud function

In the emulator, click on `Firestore.` Click `+ Start collection`. For the `Collectionn ID` enter `Messages`. The `Document ID` will be randomly generated. In "Field`, enter `original`. `Type` should be `string`. In `Value` enter any text you want. Click `Save`. You should see your message in the database, and in a few seconds you should see the same message in UPPERCASE.

## Logs

Click `Logs`. You should see

```
11:09:10 I
function[us-central1-MakeUppercase]
Beginning execution of "MakeUppercase"
11:09:10 I
function[us-central1-MakeUppercase]
{
  "severity": "INFO",
  "message": "Uppercasing Lup1yAK4SRKLZLy4lBnQ uppercase me"
}
11:09:10 I
function[us-central1-MakeUppercase]
Finished "MakeUppercase" in 279.713053ms
11:09:15 W
function[us-central1-MakeUppercase]
Your function timed out after ~60s. To configure this timeout, see
      https://firebase.google.com/docs/functions/manage-functions#set_timeout_and_memory_allocation.
11:09:15 W
function[us-central1-MakeUppercase]
Your function was killed because it raised an unhandled error.
```

`I` means `INFO`, i.e., everything is good. `W` means `WARNING`. I have no idea why every function ends with a timed out error. I presume this is a bug in the emulators.

## Storage


