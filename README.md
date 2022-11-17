# Writing Firebase Cloud Functions on the Firebase Emulator

This tutorial will show you how to write one simple Firebase Cloud Function that reads and writes to Firestore using the Firebase Local Emulator Suite.

The [official documentation](https://firebase.google.com/docs/emulator-suite) is good. This tutorial covers items left out of the official documentation.

## Why Use the Firebase Local Emulator Suite?

You don't have to use the Emulator. You could deploy your Cloud Functions to the Firebase Cloud, trigger them and see the results logged in your Firebase Console. Deploying a function takes a few minutes. You should then wait a minute for your code to propagate. Waiting for the results to log in the console takes a few minutes. Looking at my logs, my cycle time for writing code, deploying, running the function, looking at the results, fixing my code, and deploying the updated code is ten minutes. With emulator this cycle time is reduced to one minute. That's a 10x improvement in development time!

Another reason to use the emulator is that infinite loops won't cost you anything. If you mistakenly deploy an infinite loop to the cloud, Google will bill you for the computing time. And there's no way to kill a function other than deploying updated code. Your infinite loop will likely run for ten minutes.

## Which Emulator(s) To Use?

The Firebase Local Emulator Suite consists of seven emulators: Auth, Realtime Database, Firestore, Storage, and, in beta, Functions, Pub/Sub, and Extentions. This tutorial will just use Functions and Firestore.

The official documentation mentions several [other tools for prototypes and testing](https://firebase.google.com/docs/emulator-suite#other_tools_for_prototyping_and_testing). I haven't tried these, they might work better for some stuff.

## Initialize the Emulator

Make a new directory.

```
mkdir EmulatorTutorial
cd EmulatorTutorial
```

Follow the [Getting Started](https://firebase.google.com/docs/emulator-suite/connect_and_prototype) instructions.







Initialize the emulator(s).

```
firebase init emulator
```

