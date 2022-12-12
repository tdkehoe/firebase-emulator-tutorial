# Writing Firebase Cloud Functions with the Firebase Emulator and Angular.

This tutorial will show you how to write Triggerable and Callable Firebase Cloud Functions using the Firebase Local Emulator Suite. We'll learn how to call and trigger the functions from Angular.

There is official documentation for the [Emulator](https://firebase.google.com/docs/emulator-suite) and for [Cloud Functions](https://firebase.google.com/docs/functions).

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

Select one of your existing Firebase projects, create a new project, or make a demo project.

### Use a demo project

This documentation describes [Firebase demo projects](https://firebase.google.com/docs/emulator-suite/connect_firestore#choose_a_firebase_project). A demo project is preferred for codelabs or tutorials. A demo project ID must start with `demo-`, for example, `demo-test` or `demo-tutorial`. To make a demo project, start the emulator with the `--project` flag:

```
firebase emulators:start --project demo-test
```

You can skip the next few sections.

### Use a real project

You may prefer to use a real project. View which project your local app is connected to:

```
firebase use
```

If you need to change this, this command has three options. To hook up your project `my-project-123`:

```
firebase use my-project-123
```

To clear the active project:

```
firebase use clear
```

To remove a project from your app:

```
firebase use --unalias my-project-123
```

More [documentation](https://firebase.google.com/docs/cli#use_aliases) on `firebase use`.

### Connect your functions to Firebase

Find your credentials in the Firebase console. Put these in `environments.ts`.

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
```

### Start Emulator

Gentlemen, start your emulators!

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

export const MakeUppercase = functions.firestore.document('Messages/{docId}').onCreate((snap, context) => {
  try {
    const original = snap.data().original;
    const uppercase = original.toUpperCase();
    return snap.ref.set({ uppercase }, { merge: true });
  } catch (error) {
    console.error(error);
  }
});
```

This function is a little different from the example code in the documentation. They set up a CommonJS module (`require`), I set up an ES module (`import`).

## Trigger your cloud function

In the emulator, click on `Firestore.` Click `+ Start collection`. For the `Collection ID` enter `Messages`. The `Document ID` will be randomly generated. In "Field`, enter `original`. `Type` should be `string`. In `Value` enter any text you want. Click `Save`. You should see your message in the database, and in a few seconds you should see the same message in UPPERCASE.

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

### Logs

Let's add logs, two ways:

```js
console.log(context.params.docId);
functions.logger.log('Uppercasing', context.params.docId, original);
```

### Write to another collection

I can't figure out how to write to another Firestore collection in the emulator.

Change this line

```js
return snap.ref.set({ uppercase }, { merge: true }); // writes to the same document
```

to

```js
return setDoc(
    doc(db, 'AnotherCollection', context.params.docId),
    { uppercase }, { merge: true }
);
```

### Write to Cloud Firestore

In `environment.ts` change `production` to `true` to write to a Cloud Firestore project. That won't work if you're using a `demo-` project ID.

## Storage

Add these lines to your `index.js`:

```js
import { getStorage, connectStorageEmulator } from "firebase/storage";

const storage = getStorage();
connectStorageEmulator(storage, "localhost", 9199);
```

I can't figure out how to write to the Storage emulator.

```
return uploadString(storageRef, uppercase).then((snapshot) => {
    console.log('Uploaded a raw string!');
});
```

That code throws this error:

```
TypeError: storage.collection is not a function
```

## Trigger your function from Angular

I can't get a function in the emulator to trigger from Angular.

Import AngularFire modules:

*app.module.ts*
```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { environment } from '../environments/environment';

// Angular
import { FormsModule } from '@angular/forms';

// AngularFire
import { provideFirebaseApp, initializeApp } from '@angular/fire/app';
import { provideFirestore, getFirestore, connectFirestoreEmulator } from '@angular/fire/firestore';
import { provideFunctions,getFunctions, connectFunctionsEmulator } from '@angular/fire/functions';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    provideFirebaseApp(() => initializeApp(environment.firebaseConfig)),
    provideFirestore(() => {
      const firestore = getFirestore();
      if (!environment.production) {
        connectFirestoreEmulator(firestore, 'localhost', 8080);
      }
      return firestore;
    }),
    provideFunctions(() => {
      const functions = getFunctions();
      if (!environment.production) {
        connectFunctionsEmulator(functions, 'localhost', 5001);
      }
      return functions;
    }),
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

In the view, make a form.

*app.component.html*
```html
<h3>Trigger Firebase Cloud Function</h3>
<form (ngSubmit)="triggerMe(triggerText)">
    <input type="text" [(ngModel)]="triggerText" name="message" placeholder="message" required>
    <button type="submit" value="Submit">Submit</button>
</form>
```

Write the handler.

*app.component.ts```
```js
import { Component } from '@angular/core';
import { getFunctions, httpsCallable, httpsCallableFromURL } from "firebase/functions";
import { initializeApp } from 'firebase/app';
import { getFirestore, setDoc, doc } from "firebase/firestore";
import { environment } from '../environments/environment';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  firebaseConfig = environment.firebaseConfig;
  firebaseApp = initializeApp(this.firebaseConfig);
  firestore = getFirestore(this.firebaseApp);
  functions = getFunctions(this.firebaseApp);

  triggerText: string = "";
  messageText: string = "";

  async triggerMe(triggerText: string) {
    try {
      console.log("Triggering Cloud Function: " + triggerText);
      await setDoc(doc(this.firestore, "Messages", triggerText), {
        original: triggerText,
        createdAt: new Date()
      });
    } catch(error) { 
      console.error("Error adding document: ", error);
    }
  };
}
```

This throws a security rules error:

```
Error adding document:  FirebaseError: Missing or insufficient permissions.
```

The Firestore Emulator doesn't have security rules. My guess is that it rejects every read and write from outside the emulator or, more specifically, from outside the server running on `127.0.0.1:8080`.

### Trigger your function on Cloud Firestore

When you deploy your function to Cloud Firebase you can trigger it from Angular writing to your Cloud Firestore. This is straightforward and reliable. I have a tutorial that explains how to write from [Angular to Firestore using AngularFire](https://github.com/tdkehoe/Using-Firebase-with-Angular-and-AngularFire).

# Call your function from Angular

To make a callable function use `onCall`:

*index.js*
```js
export const addMessage = functions.https.onCall((data, context) => {
  try {
    const original = data.text;
    const uppercase = original.toUpperCase();
    functions.logger.log('addMessage', original, uppercase);
    return uppercase;
  } catch (error) {
    console.error(error);
  }
});
```

The handler functions use `httpsCallableFrom URL`:

*app.component.ts*
```js
  callMe(messageText: string | null) {
    console.log("Calling Cloud Function: " + messageText);
    // const addMessage = httpsCallable(this.functions, 'addMessage'); // throws CORS error
    const addMessage = httpsCallableFromURL(this.functions, 'http://localhost:5001/demo-test/us-central1/addMessage');
    addMessage({ text: messageText })
      .then((result) => {
        console.log(result.data)
      });
  };
```

Let's take a closer look at that URL. The first part is the location of the Functions emulator.

```
┌───────────┬────────────────┬─────────────────────────────────┐
│ Emulator  │ Host:Port      │ View in Emulator UI             │
├───────────┼────────────────┼─────────────────────────────────┤
│ Functions │ 127.0.0.1:5001 │ http://127.0.0.1:4000/functions │
├───────────┼────────────────┼─────────────────────────────────┤
│ Firestore │ 127.0.0.1:8080 │ http://127.0.0.1:4000/firestore │
├───────────┼────────────────┼─────────────────────────────────┤
│ Storage   │ 127.0.0.1:9199 │ http://127.0.0.1:4000/storage   │
└───────────┴────────────────┴─────────────────────────────────┘
```

Next is the Project ID. Next is the location of the Firebase server. The last part is the name of the Cloud Function.

There's another keyword, `httpsCallable`:

```js
const addMessage = httpsCallable(this.functions, 'addMessage'); // throws CORS error
```

This uses only the name of the Cloud Function. I get a CORS errors message. Presumably it'll work if you're running your Cloud Functions and your app on the same server.

Make a form in the view.

*app.component.html*
```html
<h3>Call Firebase Cloud Function</h3>
<form (ngSubmit)="callMe(messageText)">
    <input type="text" [(ngModel)]="messageText" name="message" placeholder="message" required>
    <button type="submit" value="Submit">Submit</button>
</form>
```

## Comparing Triggerable and Callable Cloud Functions

Callable Cloud Functions are neat. The code is short and the data is returned as a promise. You can use Callable Cloud Functions in the emulator, called from Angular. This speeds up development time.

Triggerable Cloud Functions require an Observer to get the data back, making your code more complex. I haven't done a speed test but I'm sure that writing to the database to trigger the function, then writing the results to the database, then waiting for the Observer will be slower. You can't trigger a function in the emulator from Angular so you'll have to use Cloud Firestore, which slows development time to a crawl.

On the other hand, writing to Firestore should be something you've done a million times and feel comfortable with. This makes Triggerable Cloud Functions solid and reliable. Callable Cloud Functions rely on relationships between servers, which seem less reliable.

