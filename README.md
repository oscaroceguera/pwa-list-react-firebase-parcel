### Install

```shell
$ npm init -y
$ npm i -S firebase react react-dom
$ npm i -D parcel parcel-bundler
$ npm install -g firebase-tools
```

### Init Firebase

- Run `firebase init`
- Using your spacebar, select both Firestore and Hosting and hit enter
- Select Use an existing project and hit enter
- Choose the newly created project from the list and hit enter
- Keep hitting enter until you get the question Configure as a single-page app (rewrite all urls to /index.html)?. Type y and hit enter

Some files will be automatically generated for you. Open firebase.json and replace the contents with the following:

```json
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "hosting": {
    "headers": [
      {
        "source": "/serviceWorker.js",
        "headers": [
          {
            "key": "Cache-Control",
            "value": "no-cache"
          }
        ]
      }
    ],
    "public": "build",
    "ignore": ["firebase.json", "**/.*", "**/node_modules/**"],
    "rewrites": [
      {
        "source": "**",
        "destination": "/index.html"
      }
    ]
  }
}
```

### Set up Firebase Context

If you don’t have experience with the React Context API, here’s a great tutorial that explains it in detail. It simply allows us to pass data from a parent component down to a child component without using props. This becomes very useful when working with children nested in multiple layers.

Inside the src folder, create another folder called firebase and create the following files:

config.js
index.js
withFirebase.jsx

### Open `public/index.html` and replace the contents with the following:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Lists PWA</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <script src="../src/index.js"></script>
  </body>
</html>
```

### Converting the app to a progressive web app

If you’ve followed this tutorial up to this point, you’re a rockstar ⭐ and you deserve a gold medal. We’ve done most of the hard work creating the actual app, and all that’s left now is to convert it to a PWA and make it work offline.

But to do this, we need to understand two key components of PWAs:

- Web app manifests
- Service workers

Inside the public folder, create a new file called manifest.webmanifest and paste the following content inside:

```
{
  "name": "Lists PWA",
  "short_name": "Idea!",
  "icons": [
    {
      "src": "./icons/icon-128x128.png",
      "type": "image/png",
      "sizes": "128x128"
    },
    {
      "src": "./icons/icon-256x256.png",
      "type": "image/png",
      "sizes": "256x256"
    },
    {
      "src": "./icons/icon-512x512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "background_color": "#333",
  "theme_color": "#39c16c",
  "orientation": "portrait"
}
```

Notice that in the "icons" section, we are linking to .png files under a /icons folder. You can get these images from the GitHub repo here, or you could choose to use custom images. Every other thing should be self-explanatory.

Now let’s make some changes to the index.html file. Open the file and add the following to the <head> section:

```html
<link rel="shortcut icon" href="icons/icon-128x128.png" />
<link rel="manifest" href="manifest.webmanifest" />
<link rel="apple-touch-icon" href="icons/icon-512x512.png" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="black" />
<meta name="apple-mobile-web-app-title" content="Lists PWA" />
<meta name="theme-color" content="#39c16c" />
<meta name="description" content="Lists PWA with React" />
```

Here’s what is going on:

- We add a shortcut icon to be displayed in our browser tab header
- We link to the manifest file we just created
- Because Safari on iOS doesn’t support the web app manifest yet, we add some traditional meta tags to make up for it (anything prefixed with apple)
- We add a theme color to theme the browser’s address bar to match our preferred brand color
- Lastly, we add a short description of our app

### Registering the service worker

Under the public folder, create a new file called serviceWorker.js and paste in the following for now: console.log('service worker registered').

Now open the index.html file and add a new script:

```html
<script>
  if ("serviceWorker" in navigator) {
    window.addEventListener("load", () => {
      navigator.serviceWorker.register("serviceWorker.js");
    });
  }
</script>
```

So this is what we need to do:

- When the SW is installed, cache all the files required for the app to work offline
- When we receive any GET network requests, we will try to respond with live data, and if that fails (due to a lack of network connection), we’ll respond with our cached data

Caching the required files
Open the serviceWorker.js file and replace the contents with the following:

```javascript
const version = "v1/";
const assetsToCache = [
  "/",
  "/src.7ed060e2.js",
  "/src.7ed060e2.css",
  "/manifest.webmanifest",
  "/icon-128x128.3915c9ec.png",
  "/icon-256x256.3b420b72.png",
  "/icon-512x512.fd0e04dd.png"
];

self.addEventListener("install", event => {
  self.skipWaiting();

  event.waitUntil(
    caches
      .open(version + "assetsToCache")
      .then(cache => cache.addAll(assetsToCache))
      .then(() => console.log("assets cached"))
  );
});
```

What’s going on here? Well, at the beginning, we’re defining two variables:

- version: Useful for keeping track of your SW version
- assetsToCache: The list of files we want to cache. These files are required for our application to work properly

**Note** : The following section only applies if you use Parcel to bundle your application.
