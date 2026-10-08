/* =========================================================
   sw.js — Service Worker
   ========================================================= */

const CACHE_VERSION = 'v20.0.0';
const CACHE_NAME = `collect-app-${CACHE_VERSION}`;
const RUNTIME_CACHE = `collect-runtime-${CACHE_VERSION}`;

const PRECACHE_URLS = [
  '/',
  '/index.html',
  '/admin.html',
  '/calculator.html',
  '/manifest.json',
  '/icon-192.png',
  '/icon-512.png',
  'https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.2.0/crypto-js.min.js',
  'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js',
  'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js',
  'https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js'
];

/* ============ Install ============ */
self.addEventListener('install', event => {
  console.log('📦 SW: Installing...');
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => Promise.allSettled(
        PRECACHE_URLS.map(url =>
          cache.add(url).catch(err =>
            console.warn('⚠️ فشل تخزين:', url)
          )
        )
      ))
      .then(() => {
        console.log('✅ SW: Installed');
        return self.skipWaiting();
      })
  );
});

/* ============ Activate ============ */
self.addEventListener('activate', event => {
  console.log('🔄 SW: Activating...');
  event.waitUntil(
    caches.keys()
      .then(keys => Promise.all(
        keys
          .filter(k => k !== CACHE_NAME && k !== RUNTIME_CACHE)
          .map(k => {
            console.log('🗑️ حذف كاش قديم:', k);
            return caches.delete(k);
          })
      ))
      .then(() => {
        console.log('✅ SW: Activated');
        return self.clients.claim();
      })
  );
});

/* ============ Fetch ============ */
self.addEventListener('fetch', event => {
  const { request } = event;
  const url = new URL(request.url);

  if(request.method !== 'GET') return;
  if(url.protocol === 'chrome-extension:') return;
  if(url.hostname.includes('google-analytics')) return;

  // CDN: Cache First
  if(url.hostname === 'cdnjs.cloudflare.com' ||
     url.hostname === 'fonts.googleapis.com' ||
     url.hostname === 'fonts.gstatic.com'){
    event.respondWith(
      caches.match(request).then(cached => {
        if(cached) return cached;
        return fetch(request).then(response => {
          const clone = response.clone();
          caches.open(RUNTIME_CACHE).then(c => c.put(request, clone));
          return response;
        }).catch(() => cached);
      })
    );
    return;
  }

  // Network First
  event.respondWith(
    fetch(request)
      .then(response => {
        if(response.ok && response.type === 'basic'){
          const clone = response.clone();
          caches.open(CACHE_NAME).then(c => c.put(request, clone));
        }
        return response;
      })
      .catch(() => {
        return caches.match(request).then(cached => {
          if(cached) return cached;
          if(request.mode === 'navigate'){
            return caches.match('/index.html');
          }
          return new Response('غير متصل بالإنترنت', {
            status: 503,
            statusText: 'Offline',
            headers: new Headers({
              'Content-Type': 'text/plain; charset=utf-8'
            })
          });
        });
      })
  );
});

/* ============ Message ============ */
self.addEventListener('message', event => {
  if(event.data === 'skipWaiting'){
    self.skipWaiting();
  }
  if(event.data === 'clearCache'){
    caches.keys().then(keys => {
      keys.forEach(key => caches.delete(key));
    });
  }
});