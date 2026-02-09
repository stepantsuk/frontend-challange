# HTML Drag and Drop

**HTML Drag and Drop** - это нативное браузерное API для реализации перетаскивания элементов, основанное на системе событий. Не требует дополнительных библиотек.
___
## I Процесс
1. Делаем элемент перетаскиваемым:
Атрибут draggable="true" (булево значение).
Важно: Для ссылок (`<a>`) и изображений (`<img>`) значение по умолчанию true. Для всего остального — false.

2. Две стороны процесса:
Источник (Drag Source): Элемент, который тащат. Отвечает за начало операции и передачу данных.
Цель (Drop Target): Область, куда можно бросить элемент. Должна "разрешить" на себе дроп.

3. Объект DataTransfer:
Единственный способ передачи данных между источником и целью.
Хранит данные в виде пар (type, data).
Доступен во всех событиях DnD как event.dataTransfer.
___
## II Ключевые события (в порядке срабатывания)
- На Источнике (Drag Source):
dragstart (КРИТИЧНО) — Срабатывает при начале перетаскивания. Здесь обязательно устанавливаем данные в dataTransfer.setData().
drag — Срабатывает постоянно, пока элемент тащат (как mousemove).
dragend — Срабатывает при отпускании элемента (успешно или нет). Место для очистки, отправки запроса и т.д.

- На Цели (Drop Target):
dragenter — Курсор с элементом впервые заходит на цель. Здесь нужно предотвратить действие по умолчанию (event.preventDefault()), чтобы цель стала "дропабельной".
dragover (КРИТИЧНО) — Курсор с элементом находится над целью. Срабатывает постоянно. Также обязательно требует preventDefault() для разрешения дропа.
dragleave — Курсор покидает цель.
drop (КРИТИЧНО) — Элемент отпущен на цели. Здесь извлекаем данные (dataTransfer.getData()) и выполняем основную логику (перенос, копирование). Тоже требует preventDefault().

_**Простая мнемоника для цели**_: Чтобы цель заработала, preventDefault() в dragenter и dragover.
___
## III DataTransfer — главный инструмент

_Методы_:
setData(format, data) — Установка данных. format — это MIME-type (например, 'text/plain', 'application/json'). Можно вызвать несколько раз с разными форматами.
getData(format) — Получение данных. Формат должен совпадать с установленным.
clearData() — Очистка.
setDragImage(image, xOffset, yOffset) — Задание кастомного изображения при drag (вместо полупрозрачного скриншота элемента).

_Свойства_:
effectAllowed (на источнике) и dropEffect (на цели) — Управляют видом курсора и логикой операции ('copy', 'move', 'link', 'none').
files — Доступ к перетаскиваемым файлам (при drag из файловой системы). Ключевая фишка API для загрузки файлов.

Проблема классического подхода:
setData()/getData() работают только со строками, что неудобно для передачи объектов. Приходится использовать JSON.stringify/parse:
```js
// Старый подход
e.dataTransfer.setData('application/json', JSON.stringify({ id: 1, name: 'Item' }));
// В drop:
const data = JSON.parse(e.dataTransfer.getData('application/json'));
```

Современная альтернатива: DataTransferItemList
В современных браузерах (с поддержкой DataTransferItem) появилась более гибкая система через свойство items:

```js
// В dragstart можно добавить данные разных типов
e.dataTransfer.items.add('text/plain', 'Hello World');
e.dataTransfer.items.add('application/json', JSON.stringify({ id: 123 }));
```
Также можно использовать WeakMap для создания приватного хранилища
___
### Краткий пример кода (ванильный JS) для запоминания:
```javascript
// 1. Делаем элемент draggable
draggableItem.setAttribute('draggable', 'true');

// 2. Источник: Начали тащить
draggableItem.addEventListener('dragstart', (e) => {
e.dataTransfer.setData('text/plain', draggableItem.id);
e.dataTransfer.effectAllowed = 'move';
});

// 3. Цель: Разрешаем дроп
dropZone.addEventListener('dragover', (e) => {
e.preventDefault(); // Обязательно!
e.dataTransfer.dropEffect = 'move';
});

// 4. Цель: Логика при дропе
dropZone.addEventListener('drop', (e) => {
e.preventDefault();
const id = e.dataTransfer.getData('text/plain');
const draggedElement = document.getElementById(id);
dropZone.appendChild(draggedElement);
});
```
___
### React библиотеки
react-dnd, react-beautiful-dnd
___

## Вывод
**HTML5 Drag and Drop API** — это нативное решение для перетаскивания, основанное на событиях. Его ядро — это работа с объектом DataTransfer для обмена данными и правильная обработка двух цепочек событий (на источнике и цели), где критически важно использовать preventDefault() на цели. Несмотря на возможность создать базовый функционал, API имеет ограничения по кастомизации и работе со сложными данными, поэтому в больших React-приложениях часто используют специализированные библиотеки, которые предоставляют более декларативный и интегрированный с React подход.


