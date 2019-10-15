# Progressive Web App Example

This repo explores some Progressive Web App techniques to build a weather app.
It is an extract from [this Google Developers tutorial][codelab]. It uses a service
worker to precache app resources, and caches weather data at runtime.

## To Run

Register for an API key with [Darksky][darksky] and store in .env file as a constant named `DARKSKY_API_KEY`
`npm install`
`node server.js`

It is recommended to install the Lighthouse Chrome Extension, which can be used to audit the page and generate
a report on how well the page measures up in regards to Progressive Web App features and Search Engine Optimisation.

[codelab]: https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/
[darksky]: https://darksky.net/dev

## Notes

# The App Shell Concept

This app shell approach is a combination of SSR and CSR, and involves loading a minimal user interface as soon as
possible, caches it for offline visits, then loads all contents of the app.

# Progressive Enhancement

Features were progressively added whilst ensuring that the site still works if the user's browser does not support them. This
approach is known as 'progressive enhancement'.

# Service Workers

A service worker is a fundamental part of a progressive web app. It is a JavaScript file that runs on a separate thread from the webpage
and can be used to do such tasks as take control of network requests, modify them, and serve custom responses from the cache. The Service
Worker lifecycle consists of the following stages:

- Registration: If supported, it tells your browser where the service worker is located and to start installing it in the background
- Instillation: Triggers an install event where we can run some tasks when the service worker is installed
- Activation: Deletes any files that are no longer necessary and cleans up

# Service Worker Code examples

Registration (see `index.html`). Registers `service-worker.js`:

```
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js').then(reg => {
      console.log('Service worker registered.', reg);
    });
  });
}
```

Installation (see `service-worker.js`) Precaches files and assets:

```
self.addEventListener('install', e => {
  e.waitUntil(
    caches.open(CACHE_NAME).then(cache => {
      return cache.addAll(FILES_TO_CACHE);
    })
  );

  self.skipWaiting();
});
```

Responding to fetches (see `service-worker.js`). Interacepts requests to the weather api and
stores their responses in the cache. Otherwise handles requests for files or assets:

```
self.addEventListener('fetch', evt => {
  if (evt.request.url.includes('/forecast/')) {
    evt.respondWith(
      caches.open(DATA_CACHE_NAME).then(cache => {
        return fetch(evt.request)
          .then(response => {
            if (response.status === 200) {
              cache.put(evt.request.url, response.clone());
            }
            return response;
          })
          .catch(err => {
            return cache.match(evt.request);
          });
      })
    );
    return;
  }
  evt.respondWith(
    caches.open(CACHE_NAME).then(cache => {
      return cache.match(evt.request).then(response => {
        return response || fetch(evt.request);
      });
    })
  );
});
```

The tutorial also goes on to show how to enable the site to make it installable on a user's device. This has
not been included to avoid cluttering the examples.
