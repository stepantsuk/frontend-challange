# VisualViewport

Предоставляет информацию о видимой области просмотра (visible part of the page), которая может отличаться от layout
viewport при:

- Масштабировании (пинч-зум)
- Появлении виртуальной клавиатуры
- Сворачивании панелей браузера
- Динамических оверлеях (поиск, панели)

**layout viewport** - это область на которой размещены все элементы страницы
**visual viewport** - это часть layout viewport-а, которая отображается в данный момент на экране.
**viewport** - задает размер layout viewport-а, для того чтобы при открытии на мобильном устройстве, ширина viewport-a
была задана наиболее удобным значением, обычно это device-width. В этом случае страница будет вписана в размер экрана.
___

## Ключевые отличия от обычного Viewport

1. Layout Viewport (document.documentElement)

- Фиксированные размеры (ширина/высота документа)
- Не меняется при масштабировании
- Событие resize на window

2. Visual Viewport (window.visualViewport)

- Видимая область прямо сейчас
- Изменяется при зуме, скролле, появлении клавиатуры
- Свойства в CSS пикселях (не device pixels)

___

## Основные свойства объекта visualViewport

```js
const vv = window.visualViewport;
// Основные метрики (в CSS пикселях)
vv.width;      // Ширина видимой области
vv.height;     // Высота видимой области
vv.scale;      // Текущий масштаб (1.0 = 100%)
vv.offsetLeft; // Смещение слева относительно layout viewport
vv.offsetTop;  // Смещение сверху

// Позиция прокрутки видимой области
vv.pageLeft;   // Горизонтальный скролл (аналог window.scrollX)
vv.pageTop;    // Вертикальный скролл (аналог window.scrollY)
```

___

## События визуального вьюпорта

1. resize - изменение размеров видимой области

```js

visualViewport.addEventListener('resize', () => {
  console.log('Новые размеры:', visualViewport.width, visualViewport.height);
  console.log('Масштаб:', visualViewport.scale);
});
```

2. scroll - прокрутка видимой области
```js
   visualViewport.addEventListener('scroll', () => {
   console.log('Позиция:', visualViewport.pageLeft, visualViewport.pageTop);
   });
```
3. Обновление UI при изменениях
```javascript
   function updateUI() {
  // Фиксируем элемент относительно видимой области
  const button = document.querySelector('.floating-button');
  button.style.bottom = '20px';
  button.style.right = `${20 / visualViewport.scale}px`; // Компенсация масштаба
}

visualViewport.addEventListener('resize', updateUI);
visualViewport.addEventListener('scroll', updateUI);
```
Более подробно тут https://chat.deepseek.com/a/chat/s/650ae0b4-6aef-43dd-b877-4fbcf964f0d4

