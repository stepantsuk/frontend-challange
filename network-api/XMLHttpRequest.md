# XMLHttpRequest

## Главное
XMLHttpRequest — это legacy API для AJAX-запросов, которое сейчас заменено Fetch API. Ключевые особенности: событийная модель с readyState, встроенная поддержка отслеживания прогресса загрузки и отправки, возможность синхронных запросов (только в воркерах). Основное современное применение — загрузка файлов с индикацией прогресса и поддержка старых браузеров. Для нового кода следует использовать Fetch API с AbortController для отмены и Streams API для прогресса.
___
### Базовый паттерн использования
```js
const xhr = new XMLHttpRequest(); // 1. Создание экземпляра

// 2. Инициализация запроса
xhr.open('GET', '/api/data', true); // true = асинхронно

// 3. Настройка обработчиков
xhr.onload = function() {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log('Успех:', xhr.responseText);
  } else {
    console.error('Ошибка:', xhr.statusText);
  }
};

xhr.onerror = function() {
  console.error('Ошибка сети');
};

// 4. Отправка запроса
xhr.send();
```
___
### Ключевые свойства и методы
**Основные методы:**
```js
xhr.open(method, url, async, user, password); // Инициализация
xhr.send(body); // Отправка (body: string, FormData, Blob, Document)
xhr.abort(); // Отмена запроса
xhr.setRequestHeader(name, value); // Установка заголовков
xhr.getResponseHeader(name); // Получение заголовка ответа
xhr.getAllResponseHeaders(); // Все заголовки ответа
```
___
### Важные свойства:
```js
// Состояние запроса
xhr.readyState; // 0-4 (UNSENT, OPENED, HEADERS_RECEIVED, LOADING, DONE)
xhr.status; // HTTP статус (200, 404, 500...)
xhr.statusText; // Текст статуса ("OK", "Not Found")

// Данные ответа
xhr.responseText; // Ответ как текст
xhr.response; // Ответ в формате, заданном responseType
xhr.responseType; // Тип ожидаемого ответа ("" (text), "json", "blob", "document", "arraybuffer")
xhr.responseXML; // Ответ как XML Document (если применимо)

// Настройки
xhr.timeout = 5000; // Таймаут в миллисекундах
xhr.withCredentials = true; // Отправка cookies при CORS
```
___
**Отслеживание прогресса загрузкиx**
```js
// Прогресс загрузки (ответ от сервера)
xhr.onprogress = function(event) {
  if (event.lengthComputable) {
    const percentComplete = (event.loaded / event.total) * 100;
    console.log(`Загружено: ${percentComplete}%`);
  }
};

// Прогресс отправки (upload)
xhr.upload.onprogress = function(event) {
  if (event.lengthComputable) {
    const percentComplete = (event.loaded / event.total) * 100;
    console.log(`Отправлено: ${percentComplete}%`);
  }
};
```
___
**Пример POST с JSON и обработкой ошибок**
```js
function makeRequest(method, url, data) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    
    xhr.open(method, url);
    xhr.setRequestHeader('Content-Type', 'application/json');
    xhr.responseType = 'json';
    
    xhr.onload = function() {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(xhr.response);
      } else {
        reject({
          status: xhr.status,
          statusText: xhr.statusText,
          response: xhr.response
        });
      }
    };
    
    xhr.onerror = function() {
      reject(new Error('Network Error'));
    };
    
    xhr.ontimeout = function() {
      reject(new Error('Request timeout'));
    };
    
    xhr.timeout = 10000; // 10 секунд
    
    xhr.send(JSON.stringify(data));
  });
}

// Использование
makeRequest('POST', '/api/users', { name: 'John' })
  .then(data => console.log(data))
  .catch(error => console.error(error));
```
