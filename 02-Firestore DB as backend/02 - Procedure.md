# 02 - Procedure

## Configure Firestore

Create the Firestore db inside the project 

https://developers.google.com/codelabs/building-a-web-app-with-angular-and-firebase?hl=it#9

Once generated, we need to add a web-app to the db for external access

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


