# Place

В задании будем делать свою версию проекта [Place](<https://en.wikipedia.org/wiki/Place_(Reddit)>) размером 256x256
Клиентский код лежит в папке `/client`, серверный — в файле `server.mjs`

Поставь зависимости и запусти сервер. Для этого перейди в директорию задачи и выполни команду `npm install`. После установки зависимостей, выполни команду `npm run dev`. После запуска, перейди по адресу [localhost:5000](http://localhost:5000)

--1. Сейчас цвета палитры захардкожены в файле `/client/picker.mjs`, cделай так, чтобы цвета запрашивались с сервера в функции `drawPalette`

--2. На сервере мы будем использовать библиотеку [ws](https://github.com/websockets/ws) для подключения по протоколу [WebSocket](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API). Она уже подключена, `WebSocket`-сервер уже правильно интегрирован с `express`, запущен и ждёт подключений по тому же порту, что и `express`.

Сделай так, чтобы `WebSocket`-сервер начал обрабатывать подключения, т.е. события `connection`, как [в примере](https://github.com/websockets/ws#simple-server)

--3. Сделай так, чтобы после подключения, новому клиенту присылался текущий массив состояния поля. Так как от сервера к клиенту будут передаваться разные сообщения, то удобно, чтобы каждое сообщение имело следующую структуру:

```javascript
{
  type: "/* action */", // тип сообщения
  payload: {
     /* ... */ // контент сообщения
  }
}
```

Через веб-сокеты можно отправлять и, соответственно, принимать данные в [разных форматах](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/send).

Нам же подойдут обычные строки. Чтобы передать объект сообщения в виде строки, его надо предварительно сериализовать, например, с помощью `JSON.stringify()`.

Как принимать и отправлять строковые сообщения уже было показано [в примере](https://github.com/websockets/ws#simple-server)

--4. Сделай так, чтобы на клиенте при получении сообщения из `WebSocket` с начальным состоянием поля, заполнялось поле. Для этого измени обработчик события на `ws` в файле `/client/index.mjs`. В него сейчас приходят события [MessageEvent](https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent). Для десериализации данных в событии используй `JSON.parse()`. Для отрисовки начального состояния используй `drawer.putArray()`

--5. Сделай так, чтобы при клике на поле, на сервер по `WebSocket` передавалось сообщение с координатами и цветом. Посылай сообщения с помощью метода [send()](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/send). Для удобства, используй тот же формат, что и в сообщениях, посылаемых с сервера

--6. Сделай так, чтобы `WebSocket`-сервер при получении сообщения с координатами и цветом валидировал его на правильность координат и цвета, а затем рассылал всем активным клиентам. Как сделать `broadcast` посмотри [в примере](https://github.com/websockets/ws#server-broadcast)

--7. Сделай так, чтобы новые пиксели не только рассылались остальным клиентам, но и добавлялись в поле, хранимое на стороне сервера. Это нужно, чтобы новые клиенты получали текущее изображение, а не начальное. Как сделаешь — проверь: открой приложение с одной вкладки браузера, нарисуй что-нибудь, затем открой со второй вкладки и убедись, что изображения совпадают

--8. Удали из обработчика клика на поле отрисовку пикселя. Для этого сделай так, чтобы при `broadcast`-е текущему клиенту тоже отправлялось сообщение. А затем сделай так, чтобы отрисовка пикселя происходила только при получении клиентом сообщения из `WebSocket`. После этого сервер будет полностью контролировать целостность поля для рисования

9. Опубликуй своё приложение на [Heroku](https://id.heroku.com/login). Для этого потребуются `heroku-cli`, и `git`, если чего-то нет, пройди [вот эти шаги](https://devcenter.heroku.com/articles/getting-started-with-nodejs#set-up).

- Авторизируйся в Heroku: `heroku login`
- Создай новое приложение в `Heroku`: `heroku create`
- Версия `nodejs` для нашего приложения уже указана в поле `engines` в `package.json`
- Закомить все изменения: `git add .`, `git commit -m "add my app"`
- Задеплой в `Heroku`: `git push heroku master`

После деплоя:

- При деплое `Heroku` для `Node.js`-приложений по умолчанию запускает `npm-script` с именем `start`
- Чтобы запускать приложение локально можно использовать `heroku local web`
- Чтобы деплоить новую версию приложения можно повторять последние два шага

--10. Сделай так, что `WebSocket` сервер принимал только те подключения, которые передали в параметрах правильный `apiKey`. Для этого измени обработчик события `upgrade`. Подробнее о нём можно прочитать [здесь](https://nodejs.org/api/http.html#http_event_upgrade).

В обработчике уже используется объект `URL`, благодаря которому можно получить значение `apiKey` из `query string`. Как это сделать посмотри [тут](https://nodejs.org/api/url.html#url_class_urlsearchparams). Для закрытия соединения, которое произошло с невалидным `apiKey` можно использовать `socket.destroy()`. Подробнее можно прочитать [здесь](https://nodejs.org/api/stream.html#stream_writable_destroy_error)

--11. \* Сделай так, чтобы после успешного `upgrade` происходила связь `ws` клиента и его `apiKey`. Сделать это можно внутри в `handleUpgrade()`. Для этого используй [WeakMap](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap), где ключами будут объекты `ws`, а значениями — связанные с ними `apiKey`. `WeakMap` не будет препятствовать сборщику мусора очищать `ws`-соединения, когда они будут закрыты

--12. \* Сделай так, чтобы у каждого `apiKey` был свой таймаут. При подключении нового `WebSocket` клиента, посылай ему сообщение с временем, когда он в следующий раз может нарисовать на поле. Текущее время можно получить с помощью `new Date()`. Для отправки сериализуй дату с помощью `.toISOString()`.

На клиенте при получении сообщения, обновляй таймер с помощью `timeout.next =`. Десериализуй полученную дату с помощью `new Date(isoDateString)`. На сервере, при получении сообщения с координатами проверяй таймаут `apiKey` `ws` клиента и не принимай сообщения, которые произошли до истечения этого таймаута.

В ответ на принятые сообщения обновляй таймаут на `10s-60s` и отсылай его клиенту. В ответ на непринятые сообщения просто посылай таймаут.

> Подсказка: чтобы получить дату через `n` секунд после `date` можно использовать такое выражение: `new Date(date.valueOf() + n * 1000)`
