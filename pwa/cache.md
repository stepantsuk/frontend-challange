# CASHE

___

## Суть: Что такое Cache API?

Это постоянное хранилище пар «Запрос — Ответ» (Request/Response). Оно живет в изолированном пространстве (origin) и
доступно как из Service Worker, так и из основного окна. Не путайте его с HTTP-кэшем браузера — это независимый слой,
который мы контролируем программно.
___

## Основные методы

| Методr                       | Что делает                                    | Когда использовать                                     |
|------------------------------|-----------------------------------------------|--------------------------------------------------------|
| cache.put(request, response) | Вручную сохраняет пару.                       | Для динамического кэширования (например, после fetch). |
| cache.match(request)         | Ищет первый подходящий ответ.                 | 	Самая частая операция в fetch-событии.                |
| cache.add(url)               | Упрощённый put: сам скачивает и кладет в кэш. | Для статики в install.                                 |
| cache.addAll([url1, url2])   | То же, но для массива.                        | Стандарт для кэширования App Shell.                    |
| cache.delete(request)        | Удаляет запись.                               | Для чистки устаревшего кэша в activate.                |
| cache.keys()                 | Получает список всех ключей.                  | Для инвентаризации кэша.                               |

##  Критические нюансы
- Никакого уважения к HTTP-заголовкам. Cache API игнорирует Cache-Control, Expires и т.д. Запись живет, пока вы её сами не удалите. Это плюс (полный контроль), но и огромная ответственность.
- VARY имеет значение. Алгоритм поиска зависит от заголовка VARY у сохраненного ответа. Это неочевидно и может вызвать баги, если вы кэшируете ответы с разными Accept-Encoding или Accept-Language.
- Set-Cookie не сохраняются. Важно для безопасности. Ответ, который вы достали из кэша, никогда не установит куки на устройстве пользователя.
- Не живёт вечно. Браузер выделяет квоту. В любой момент может очистить весь кэш вашего origin (или ничего). Не храните пользовательские данные только здесь, используйте IndexedDB для критической информации.

##  Стратегии в PWA: Один Cache или несколько?

Несколько именованных кешей. Например:
- static-v1 — для App Shell (CSS, JS, логотипы). Почти не меняется, стратегия Cache First.
- images-v1 — для медиа, с ограничением по количеству.
- dynamic-v1 — для ответов от API (новости, контент), стратегия Stale-While-Revalidate или Network First.

Это позволяет в activate легко удалить старую версию static-v2, не трогая кэш с изображениями пользователя».
---
##  Cashe and ServiceWorker
sw.js
```js
// ====================================
// 1. УСТАНОВКА (INSTALL)
// ====================================
const CACHE_NAME = 'my-pwa-v1';  // «Версия кэша» — меняй при каждом деплое

self.addEventListener('install', event => {
  console.log('[SW] Install');

  // Заставляем новый SW сразу активироваться, не ждать закрытия вкладок
  self.skipWaiting();

  // КЭШИРУЕМ СТАТИКУ (App Shell)
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => {
      // Вот этот массив — файлы, которые будут доступны офлайн
      return cache.addAll([
        '/',
        '/index.html',
        '/style.css',
        '/script.js'
      ]);
    })
  );
});

// ====================================
// 2. АКТИВАЦИЯ (ACTIVATE) + УДАЛЕНИЕ СТАРОГО КЭША
// ====================================
self.addEventListener('activate', event => {
  console.log('[SW] Activate');

  event.waitUntil(
    // Получаем ВСЕ имена кэшей этого сайта
    caches.keys().then(keys => {
      return Promise.all(
        keys
          .filter(key => key !== CACHE_NAME) // удаляем всё, кроме текущего
          .map(key => caches.delete(key))
      );
    }).then(() => {
      // Немедленно захватываем контроль над ВСЕМИ открытыми вкладками
      return clients.claim();
    })
  );
});

// ====================================
// 3. ПЕРЕХВАТ ЗАПРОСОВ (FETCH) — ПРИМИТИВНАЯ СТРАТЕГИЯ CACHE FIRST
// ====================================
self.addEventListener('fetch', event => {
  console.log('[SW] Fetch:', event.request.url);

  event.respondWith(
    caches.match(event.request).then(cachedResponse => {
      if (cachedResponse) {
        return cachedResponse; // отдаём из кэша
      }
      return fetch(event.request); // идём в сеть
    })
  );
});
```

---
##  Cache API
```js

// ==================== CacheStorage (глобальный объект caches) ====================

// Открыть (или создать) именованный кэш
const cache = await caches.open('my-static-v1');

// ==================== Добавление записей ====================

// 1. Простое: fetch + put одной командой (только GET)
await cache.add('/index.html');
await cache.addAll(['/app.js', '/app.css']);

// 2. Ручное сохранение (полный контроль)
const request = new Request('/api/user');
const response = await fetch(request);
await cache.put(request, response.clone()); // clone — response можно использовать только раз

// ==================== Чтение ====================

// Найти первый подходящий ответ
const cachedResponse = await cache.match('/index.html');
if (cachedResponse) {
  const html = await cachedResponse.text();
}

// Найти все ответы, подходящие под условие
const options = { ignoreSearch: true, ignoreVary: true };
const allResponses = await cache.matchAll('/images/logo.jpg', options);

// ==================== Удаление ====================

// Удалить одну запись
const deleted = await cache.delete('/old-style.css');

// Удалить целый CacheStorage
await caches.delete('my-old-v1');

// ==================== Получение ключей ====================

// Все запросы (ключи) в текущем кэше
const keys = await cache.keys();
keys.forEach(request => console.log(request.url));

// Все имена кэшей в origin
const cacheNames = await caches.keys();
console.log(cacheNames); // ['my-static-v1', 'my-dynamic-v2']

// ==================== Очистка старых версий (типичный паттерн) ====================

const activeCaches = ['static-v3', 'dynamic-v2'];
const allCaches = await caches.keys();
await Promise.all(
  allCaches
    .filter(name => !activeCaches.includes(name))
    .map(name => caches.delete(name))
);
```
