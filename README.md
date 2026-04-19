# Simple Web App To PWA Lab Practical

This project is a simple web app converted into a Progressive Web App using a web app manifest, service worker, static caching, dynamic caching, and offline fallback support.

## Project Structure

```text
pwa/
  index.html
  manifest.json
  sw.js
  css/
    styles.css
    materialize.min.css
  js/
    app.js
    ui.js
    materialize.min.js
  img/
    dish.png
    icons/
      icon-72x72.png
      icon-96x96.png
      icon-128x128.png
      icon-144x144.png
      icon-152x152.png
      icon-192x192.png
      icon-384x384.png
      icon-512x512.png
  pages/
    about.html
    contact.html
    fallback.html
```

## Basic Requirements

To convert a simple web app into a PWA, the following things are required:

1. Simple web app files like `index.html`, CSS, JavaScript, and images.
2. `manifest.json` file.
3. App icons in different sizes.
4. `sw.js` service worker file.
5. Service worker registration code.
6. Static caching code.
7. Dynamic caching code.
8. Offline fallback page.
9. Browser support, preferably Google Chrome.
10. Local server or HTTPS.

Do not open the project directly using `file:///`. Service workers require `localhost` or HTTPS.

Run the project using a local server:

```bash
python -m http.server 8000
```

Then open:

```text
http://localhost:8000
```

## Experiment 1: Write `manifest.json` File And Include It In The Website

### Aim

To create a web app manifest file and include it in the website so the web app can become installable like a mobile or desktop app.

### Requirements

```text
manifest.json
index.html
app icons
```

### Steps

1. Create a file named `manifest.json` in the root folder.

2. Add app information inside `manifest.json`.

```json
{
  "name": "Food Ninja",
  "short_name": "FoodNinja",
  "start_url": "/index.html",
  "display": "standalone",
  "background_color": "#FFE9D2",
  "theme_color": "#FFE1C4",
  "orientation": "portrait-primary",
  "icons": [
    {
      "src": "/img/icons/icon-72x72.png",
      "type": "image/png",
      "sizes": "72x72"
    },
    {
      "src": "/img/icons/icon-96x96.png",
      "type": "image/png",
      "sizes": "96x96"
    },
    {
      "src": "/img/icons/icon-192x192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "/img/icons/icon-512x512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ]
}
```

3. Include the manifest file in the `<head>` section of `index.html`.

```html
<link rel="manifest" href="/manifest.json">
```

4. Add theme color in `index.html`.

```html
<meta name="theme-color" content="#FFE1C4">
```

5. Add iOS icon support.

```html
<link rel="apple-touch-icon" href="/img/icons/icon-96x96.png">
<meta name="apple-mobile-web-app-status-bar" content="#FFE1C4">
```

### Explanation

The `manifest.json` file provides information about the web app such as app name, short name, start URL, display type, theme color, background color, orientation, and icons.

### Testing

1. Open the website in Chrome.
2. Open DevTools.
3. Go to `Application` tab.
4. Click `Manifest`.
5. Check app name, icons, theme color, and start URL.

### Result

The website now contains a valid `manifest.json` file and is ready for PWA installability.

## Experiment 2: Write, Register, Install And Activate A Service Worker

### Aim

To create and register a service worker file so the web app can use PWA features such as offline support and caching.

### Requirements

```text
sw.js
js/app.js
index.html
```

### Steps

1. Create a file named `sw.js` in the root folder.

2. Add install and activate events in `sw.js`.

```js
self.addEventListener('install', evt => {
  console.log('service worker installed');
});

self.addEventListener('activate', evt => {
  console.log('service worker activated');
});
```

3. Register the service worker in `js/app.js`.

```js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('service worker registered', reg))
    .catch(err => console.log('service worker not registered', err));
}
```

4. Include `js/app.js` in `index.html` before the closing `</body>` tag.

```html
<script src="/js/app.js"></script>
```

5. Run the app using `localhost`.

```bash
python -m http.server 8000
```

6. Open:

```text
http://localhost:8000
```

### Explanation

A service worker is a JavaScript file that runs in the browser background. It can intercept network requests, cache files, and provide offline support.

The `install` event runs when the service worker is installed for the first time.

The `activate` event runs when the service worker becomes active.

### Testing

1. Open Chrome DevTools.
2. Go to `Application` tab.
3. Click `Service Workers`.
4. Check that `sw.js` is registered, activated, and running.

### Result

The service worker is successfully registered, installed, and activated.

## Experiment 3: Static Caching In The Service Worker

### Aim

To cache fixed app files during service worker installation so the app shell can load offline.

### Requirements

```text
sw.js
Cache API
static files list
```

### Static Files Examples

Static files are files that are already present in the project, such as:

```text
index.html
CSS files
JavaScript files
images
about.html
contact.html
fallback.html
```

### Steps

1. Add a static cache name in `sw.js`.

```js
const staticCacheName = 'site-static-v1';
```

2. Create a list of files to cache.

```js
const assets = [
  '/',
  '/index.html',
  '/css/styles.css',
  '/css/materialize.min.css',
  '/js/app.js',
  '/js/ui.js',
  '/js/materialize.min.js',
  '/img/dish.png',
  '/pages/about.html',
  '/pages/contact.html',
  '/pages/fallback.html'
];
```

3. Cache static files inside the `install` event.

```js
self.addEventListener('install', evt => {
  evt.waitUntil(
    caches.open(staticCacheName).then(cache => {
      console.log('caching static assets');
      return cache.addAll(assets);
    })
  );
});
```

4. Return cached files inside the `fetch` event.

```js
self.addEventListener('fetch', evt => {
  evt.respondWith(
    caches.match(evt.request).then(cacheRes => {
      return cacheRes || fetch(evt.request);
    })
  );
});
```

5. Delete old caches inside the `activate` event.

```js
self.addEventListener('activate', evt => {
  evt.waitUntil(
    caches.keys().then(keys => {
      return Promise.all(
        keys
          .filter(key => key !== staticCacheName)
          .map(key => caches.delete(key))
      );
    })
  );
});
```

### Explanation

Static caching stores important app files during service worker installation. Because these files are cached in advance, the web app can load even when the internet is not available.

### Testing

1. Open the website online.
2. Open Chrome DevTools.
3. Go to `Application` tab.
4. Click `Cache Storage`.
5. Check the static cache name.
6. Turn on offline mode from `Service Workers`.
7. Refresh the page.

### Result

The app shell files are cached and the web app can load offline.

## Experiment 4: Dynamic Caching In The Service Worker

### Aim

To cache files dynamically at runtime when the browser requests them.

### Requirements

```text
static cache
dynamic cache
fetch event
offline fallback page
cache size limit
```

### Dynamic Files Examples

Dynamic files are files that are requested while using the app, such as:

```text
new pages
API responses
external fonts
images loaded later
runtime network requests
```

### Steps

1. Add a dynamic cache name in `sw.js`.

```js
const dynamicCacheName = 'site-dynamic-v1';
```

2. Add dynamic caching code inside the `fetch` event.

```js
self.addEventListener('fetch', evt => {
  evt.respondWith(
    caches.match(evt.request).then(cacheRes => {
      return cacheRes || fetch(evt.request).then(fetchRes => {
        return caches.open(dynamicCacheName).then(cache => {
          cache.put(evt.request, fetchRes.clone());
          return fetchRes;
        });
      });
    })
  );
});
```

3. Add a fallback page at `pages/fallback.html`.

```html
<!DOCTYPE html>
<html>
<head>
  <title>Offline</title>
</head>
<body>
  <h2>You are offline</h2>
  <p>This page is not available without internet.</p>
</body>
</html>
```

4. Add fallback logic if the network fails.

```js
self.addEventListener('fetch', evt => {
  evt.respondWith(
    caches.match(evt.request).then(cacheRes => {
      return cacheRes || fetch(evt.request).then(fetchRes => {
        return caches.open(dynamicCacheName).then(cache => {
          cache.put(evt.request, fetchRes.clone());
          return fetchRes;
        });
      });
    }).catch(() => {
      if (evt.request.url.indexOf('.html') > -1) {
        return caches.match('/pages/fallback.html');
      }
    })
  );
});
```

5. Add a function to limit dynamic cache size.

```js
const limitCacheSize = (name, size) => {
  caches.open(name).then(cache => {
    cache.keys().then(keys => {
      if (keys.length > size) {
        cache.delete(keys[0]).then(() => limitCacheSize(name, size));
      }
    });
  });
};
```

6. Call the cache limit function after storing a dynamic response.

```js
limitCacheSize(dynamicCacheName, 15);
```

### Full Service Worker Code

```js
const staticCacheName = 'site-static-v1';
const dynamicCacheName = 'site-dynamic-v1';

const assets = [
  '/',
  '/index.html',
  '/css/styles.css',
  '/css/materialize.min.css',
  '/js/app.js',
  '/js/ui.js',
  '/js/materialize.min.js',
  '/img/dish.png',
  '/pages/about.html',
  '/pages/contact.html',
  '/pages/fallback.html'
];

const limitCacheSize = (name, size) => {
  caches.open(name).then(cache => {
    cache.keys().then(keys => {
      if (keys.length > size) {
        cache.delete(keys[0]).then(() => limitCacheSize(name, size));
      }
    });
  });
};

self.addEventListener('install', evt => {
  evt.waitUntil(
    caches.open(staticCacheName).then(cache => {
      console.log('caching static assets');
      return cache.addAll(assets);
    })
  );
});

self.addEventListener('activate', evt => {
  evt.waitUntil(
    caches.keys().then(keys => {
      return Promise.all(
        keys
          .filter(key => key !== staticCacheName && key !== dynamicCacheName)
          .map(key => caches.delete(key))
      );
    })
  );
});

self.addEventListener('fetch', evt => {
  evt.respondWith(
    caches.match(evt.request).then(cacheRes => {
      return cacheRes || fetch(evt.request).then(fetchRes => {
        return caches.open(dynamicCacheName).then(cache => {
          cache.put(evt.request, fetchRes.clone());
          limitCacheSize(dynamicCacheName, 15);
          return fetchRes;
        });
      });
    }).catch(() => {
      if (evt.request.url.indexOf('.html') > -1) {
        return caches.match('/pages/fallback.html');
      }
    })
  );
});
```

### Explanation

Dynamic caching stores files when they are requested by the browser. If the same file is requested again, the service worker can serve it from cache.

`fetchRes.clone()` is required because a response can be consumed only once. One copy is returned to the browser and the cloned copy is stored in cache.

The fallback page is shown when the user is offline and the requested HTML page is not available in cache.

### Testing

1. Open the website online.
2. Visit different pages.
3. Open Chrome DevTools.
4. Go to `Application` tab.
5. Click `Cache Storage`.
6. Check `site-dynamic-v1`.
7. Turn on offline mode.
8. Refresh already visited pages.
9. Open a page that is not cached and check the fallback page.

### Result

Runtime files are dynamically cached and can be loaded later, even when the user is offline.

## Final Practical Flow

Follow this order during practical:

1. Create or open the simple web app.
2. Add app icons.
3. Create `manifest.json`.
4. Link `manifest.json` in `index.html`.
5. Add theme color.
6. Create `sw.js`.
7. Register service worker in `js/app.js`.
8. Add `install` event.
9. Add `activate` event.
10. Add static cache name.
11. Add static assets list.
12. Cache static files in the `install` event.
13. Add `fetch` event.
14. Add dynamic cache name.
15. Cache runtime requests dynamically.
16. Add fallback page.
17. Add cache size limit.
18. Test using Chrome DevTools.
19. Test offline mode.

## Chrome DevTools Testing Checklist

Open DevTools and check:

```text
Application -> Manifest
Application -> Service Workers
Application -> Cache Storage
```

Check the following:

1. Manifest is loaded correctly.
2. App name and icons are visible.
3. Service worker is registered.
4. Service worker is activated and running.
5. Static cache is created.
6. Dynamic cache is created.
7. Offline mode works.
8. Fallback page appears when needed.

## Important Viva Questions

### What is a PWA?

A Progressive Web App is a web application that works like a native app using features such as manifest, service worker, caching, offline support, and installability.

### What is `manifest.json`?

`manifest.json` is a JSON file that contains app information such as app name, short name, icons, theme color, background color, start URL, and display mode.

### What is a service worker?

A service worker is a JavaScript file that runs in the background and can intercept network requests, cache files, and provide offline support.

### What is static caching?

Static caching means caching fixed app files during the service worker install event.

### What is dynamic caching?

Dynamic caching means caching files at runtime when they are requested by the browser.

### What is the Cache API?

The Cache API is a browser API used by service workers to store and retrieve network responses.

### Why do we use `fetchRes.clone()`?

We use `fetchRes.clone()` because a response can be used only once. One copy is returned to the browser and another copy is saved in cache.

### Why is `localhost` or HTTPS required?

Service workers work only on secure origins, such as HTTPS or `localhost`.

### What is an offline fallback page?

An offline fallback page is a page shown when the user is offline and the requested page is not available in cache.

## Short Conclusion

In this practical, a simple web app is converted into a Progressive Web App by adding a manifest file, registering a service worker, caching static files, caching dynamic requests, and providing offline fallback support.
