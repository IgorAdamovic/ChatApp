# ChatApp

Realtime chat application + Authentication system

## About the project 

Fully responsive, mobile-friendly chat application made for internal use of <a href="https://www.recognite.co.uk/">Recognite Ltd</a>. This app is still in development stage, but has 

## Firebase setup 

Create a project in <a href="https://console.firebase.google.com/">Firebase Console</a> (e.g. chat-app) 

Project settings -> Web settings (</>) -> Register app (chat-app) -> Add Firebase SDK -> Copy firebaseConfig and replace it in main.js

### Build

Authentication -> Get Started -> Enable Email/Password

Firestore -> Create database at Cloud Firestore (Start in test mode)

Realtime Database -> Setup Realtime Database (Start in test mode) -> Add users field inside default -rtdb (users : "")

Storage -> Create default bucket -> Create folder pictures

Go to sign up -> Register -> Login 

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
