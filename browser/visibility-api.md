# Видимость страницы API

Позволяет определить, видна ли страница пользователю в данный момент (активна ли вкладка/окно). Критически важно для
оптимизации производительности и энергопотребления.
При переключении между вкладками, web страница переходит в фоновый режим и поэтому не видна пользователю.

Page Visibility API позволяет определять, активна ли вкладка/окно браузера через document.visibilityState и событие visibilitychange. Критически важно для оптимизации: при скрытии страницы нужно приостанавливать медиа, анимации, сетевые запросы и фоновые задачи. В современных приложениях интегрируется с аналитикой, bfcache и другими API (Wake Lock, Picture-in-Picture). Особое внимание нужно уделять мобильным браузерам и восстановлению из кэша.

Несколько способов использования Page Visibility API.

На сайте есть слайдер изображений с автопрокруткой, которую можно поставить на паузу, когда пользователь перешёл на другую вкладку
Приложение выводит информацию в реальном времени, которую можно не обновлять, пока страница не видна, тем самым уменьшить количество запросов на сервер
Странице нужно понять, когда она должна быть отрисована, так что можно вести точный подсчёт количества просмотров
Сайту нужно выключить звук, когда устройство в режиме ожидания (пользователь нажал кнопку включения, чтобы погасить экран)
___

## Ключевые свойства и события

**interface** :
- Document.hidden
- Document.visibilityState
    - visible - The page content may be at least partially visible. In practice this means that the page is the foreground tab of a non-minimized window.

    - hidden - The page's content is not visible to the user, either due to the document's tab being in the background or part of a window that is minimized, or because the device's screen is off.

    - prerender - The page's content is being prerendered and is not visible to the user. A document may start in the prerender state, but will never switch to this state from any other state, since a document can only prerender once.

    - unloaded - The page is in the process of being unloaded from memory.
- Document.onvisibilitychange

```js
document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Страница стала скрытой
  } else {
    // Страница стала видимой
  }
});
```
___

## Критические нюансы

1. Когда страница считается скрытой:

- Переключение на другую вкладку
- Сворачивание окна браузера
- Переключение на другое приложение (Alt+Tab)
- Браузер в полноэкранном режиме, но не активен

Не считается: перекрытие другим окном (если оба окна браузера)

2. Производительность и батарея

```javascript
   // Плохо: постоянный polling
setInterval(() => {
  if (!document.hidden) {
    updateDashboard(); // Бесполезная нагрузка в фоне
  }
}, 1000);

// Хорошо: использовать видимость
document.addEventListener('visibilitychange', () => {
  if (!document.hidden) {
    updateDashboard();
    startLiveUpdates();
  } else {
    stopLiveUpdates();
  }
});
```

3. Проблемы с мобильными устройствами
```javascript
   // На iOS Safari есть особенности:
   document.addEventListener('visibilitychange', () => {
   // На iOS страница становится hidden при блокировке экрана
   // или переходе в домашний экран, но не всегда
   if (document.hidden) {
   // Сохраняем состояние формы
   localStorage.setItem('draft', formData);
   }
   });
```


## Cценарии использования

1. Оптимизация медиа-контента

```js
const video = document.querySelector('video');

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    video.pause();
    // Или video.currentTime = 0 для рекламы
  } else {
    video.play().catch(() => {
      // Автоплей может быть заблокирован браузером
    });
  }
});
```

2. Приостановка анимаций/игр

```js
let animationId;

function animate() {
  // Игровая логика/анимация
  animationId = requestAnimationFrame(animate);
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    cancelAnimationFrame(animationId);
    // Также приостановить физику, таймеры
  } else {
    animate();
  }
});
```

3. Управление сетевыми запросам

```js
// Отключаем фоновый polling при скрытой странице
let pollInterval;

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    clearInterval(pollInterval);
    // Приостанавливаем WebSocket соединение
    socket?.pause();
  } else {
    pollInterval = setInterval(fetchUpdates, 5000);
    socket?.resume();
  }
});
```

4. Аналитика и метрики

```js
let pageActiveTime = 0;
let visibilityStart = Date.now();

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    // Страница стала скрытой - фиксируем время активности
    pageActiveTime += Date.now() - visibilityStart;
    sendAnalytics('page_active_time', pageActiveTime);

    // Можно отправлять "прощальные" данные
    navigator.sendBeacon('/analytics', {
      event: 'page_hidden',
      timeSpent: pageActiveTime
    });
  } else {
    // Страница стала видимой - начинаем новый отсчет
    visibilityStart = Date.now();
    sendAnalytics('page_visible');
  }
});
```


___

## Современные особенности и лучшие практики

1. Навигационные API (BFCache)

```javascript
   // Страница может быть восстановлена из кэша назад/вперед
window.addEventListener('pageshow', (event) => {
  if (event.persisted) {
    // Страница загружена из bfcache
    console.log('Restored from bfcache');
  }
});
```

// Важно: слушатели visibilitychange сохраняются в bfcache
// Их не нужно переустанавливать

2. Worker API интеграция

```javascript
   // Можно передавать состояние видимости в Worker
document.addEventListener('visibilitychange', () => {
  navigator.serviceWorker.controller?.postMessage({
    type: 'VISIBILITY_CHANGE',
    visible: !document.hidden
  });
});
```

// Или использовать BroadcastChannel
const channel = new BroadcastChannel('visibility');
channel.postMessage({ visible: !document.hidden });

3. React-хук (современный подход)

```javascript
   import {useEffect, useState} from 'react';

const usePageVisibility = () => {
  const [isVisible, setIsVisible] = useState(!document.hidden);
  const [visibilityState, setVisibilityState] = useState(document.visibilityState);

  useEffect(() => {
    const handleChange = () => {
      setIsVisible(!document.hidden);
      setVisibilityState(document.visibilityState);

      // Современный подход: отправка в analytics/state manager
      gtag('event', 'visibility_change', {
        visibility_state: document.visibilityState
      });
    };

    // Оптимизация: passive listener для производительности
    document.addEventListener('visibilitychange', handleChange, {passive: true});

    return () => document.removeEventListener('visibilitychange', handleChange);
  }, []);

  return {
    isVisible,
    visibilityState,
// Дополнительные деривативные состояния
    isHidden: !isVisible,
    isPrerender: visibilityState === 'prerender'
  };
};
```

4. Progressive Enhancement подход

```javascript
   // Проверка поддержки и fallback
const getVisibilityState = () => {
  if (typeof document.hidden !== 'undefined') {
    return {
      hidden: document.hidden,
      visibilityState: document.visibilityState,
      supported: true
    };
  }

// Fallback для старых браузеров
  return {
    hidden: document.hasFocus ? !document.hasFocus() : false,
    visibilityState: document.hasFocus ? 'visible' : 'hidden',
    supported: false
  };
};
```

