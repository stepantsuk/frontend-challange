# Beacon API

Page Visibility API предоставляет события, которые вы можете отслеживать, чтобы узнать, когда страница станет видимой или скрытой, а так же возможность наблюдать текущее состояние видимости страницы.

Несколько способов использования Page Visibility API.

- На сайте есть слайдер изображений с автопрокруткой, которую можно поставить на паузу, когда пользователь перешёл на другую вкладку
- Приложение выводит информацию в реальном времени, которую можно не обновлять, пока страница не видна, тем самым уменьшить количество запросов на сервер
- Странице нужно понять, когда она должна быть отрисована, так что можно вести точный подсчёт количества просмотров
- Сайту нужно выключить звук, когда устройство в режиме ожидания (пользователь нажал кнопку включения, чтобы погасить экран)

Раньше у разработчиков были неудобные способы. Например, обработка событий blur и focus на объекте window помогала узнать когда страница становилась не активной, но это не давало возможность понять когда страница действительно скрыта от пользователя. Page Visibility API решает эту проблему.

**interface** :
- Document.hidden
- Document.visibilityState
  - visible - The page content may be at least partially visible. In practice this means that the page is the foreground tab of a non-minimized window.

  - hidden - The page's content is not visible to the user, either due to the document's tab being in the background or part of a window that is minimized, or because the device's screen is off.

  - prerender - The page's content is being prerendered and is not visible to the user. A document may start in the prerender state, but will never switch to this state from any other state, since a document can only prerender once.

  - unloaded - The page is in the process of being unloaded from memory.
- Document.onvisibilitychange

```js
//startSimulation and pauseSimulation defined elsewhere
function handleVisibilityChange() {
  if (document.hidden) {
    pauseSimulation();
  } else {
    startSimulation();
  }
}

document.addEventListener("visibilitychange", handleVisibilityChange, false);
```

