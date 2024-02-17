# 02 - Procedure

## Configure Firestore

Create the Firestore db inside the project 

https://developers.google.com/codelabs/building-a-web-app-with-angular-and-firebase?hl=it#9

Once generated, we need to add a web-app to the db for external access (_project settings -> app access_)

From the menu for the app integration copy the config

```typescript
// Your web app's Firebase configuration
const firebaseConfig = {
  apiKey: "***",
  authDomain: "recipes-ffddb.firebaseapp.com",
  databaseURL: "https://recipes-ffddb-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "recipes-ffddb",
  storageBucket: "recipes-ffddb.appspot.com",
  messagingSenderId: "***",
  appId: "***"
};
```

## Configure Angular app

https://medium.com/@christianaxtmn/adding-firebase-to-angular-17-5c5a7cf4aba8

### Add environment
It’s no longer standard to have different environments in an Angular application. 
But the CLI provides an easy way to add a production and a development environment. 
The command below creates two config files and automatically updates the `angular.json` file to handle the created environments:

```bash
$ ng g environments
CREATE src/environments/environment.ts (31 bytes)
CREATE src/environments/environment.development.ts (31 bytes)
UPDATE angular.json (3060 bytes)
```
We can now distinguish the configuration per env, for now we update both with 

```typescript
export const environment = {
    firebaseConfig: {
        apiKey: '<your-key>',
        authDomain: '<your-project-authdomain>',
        databaseURL: '<your-database-URL>',
        projectId: '<your-project-id>',
        storageBucket: '<your-storage-bucket>',
        messagingSenderId: '<your-messaging-sender-id>'
    }
};
```
Now let’s add [angularfire](https://github.com/angular/angularfire) to make use of Firebase:

`ng add @angular/fire`

When askes, we can deselect all features. 

Then we can add to the ``src/main.ts`` file (which has a different format on Angular 17).

Whe add the following to the list of providers:

```typescript
importProvidersFrom([
      provideFirebaseApp(() => initializeApp(environment.firebaseConfig)),
      provideAuth(() => getAuth()),
      provideFirestore(() => getFirestore()),
      provideStorage(() => getStorage())
    ]),
```
By running the installation script again, the following features can be added:

* authentication
* firestore
* Cloud storage
* deploy

The output will allow the app selection and will updated the files automatically



```bash
? Please select an app: angular-recipes
√ Downloading configuration data of your Firebase WEB app
UPDATE angular.json (3672 bytes)
UPDATE firebase.json (120 bytes)
UPDATE .firebaserc (195 bytes)
UPDATE src/main.ts (2084 bytes)
```

The automatic changes can be compared with the one listed.

### Set CORS settings on the Firestore db

Without those settings, if we try to access data with `HttpClient` from the app we will 
get 

```shell
Failed to load resource: the server responded with a status of 404 (Not Found)
localhost/:1 Access to XMLHttpRequest at 'https://console.firebase.google.com/project/recipes-ffddb/database/recipes-ffddb-default-rtdb/dataunits.json' 
from origin 'http://localhost:4200' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

We need to set the CORS settings on Firebase. We can do it accessing CLOUD SHELL on the project where
the Firestore DB resides

```shell
pedano_alessandro@cloudshell:~ (recipes-ffddb)$ gsutil ls
gs://recipes-ffddb.appspot.com/
gs://staging.recipes-ffddb.appspot.com/
pedano_alessandro@cloudshell:~ (recipes-ffddb)$ touch cors.json
pedano_alessandro@cloudshell:~ (recipes-ffddb)$ nano cors.json
pedano_alessandro@cloudshell:~ (recipes-ffddb)$ cat cors.json
[
  {
    "origin": ["*"],
    "method": ["GET", "POST", "DELETE", "PUT"],
    "maxAgeSeconds": 3600
  }
]
$ gsutil cors set cors.json gs://recipes-ffddb.appspot.com/
Setting CORS on gs://recipes-ffddb.appspot.com/...
```
**NOTE**: the **BUCKET NAME** used in the command is the same from ``firebaseConfig.storageBucket``


https://scientyficworld.org/how-to-connect-firebase-with-angular-project/