# Настройка отправки формы в Google Sheets

## Шаг 1. Создать таблицу

1. Откройте [Google Sheets](https://sheets.google.com) и создайте новую таблицу
2. Назовите её, например: **Паруса — Заявки**
3. В первой строке создайте заголовки столбцов:

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| Дата | Имя | Контакт | Компания | О проекте | Статус обработки |

4. Столбец **«Статус обработки»** заполняется вручную владельцем таблицы. Примеры значений: `Новая`, `В работе`, `Завершена`, `Отказ`
5. Доступ к таблице оставьте **ограниченным** (только вы) — менять настройки доступа к самой таблице не нужно

## Шаг 2. Создать скрипт

1. В таблице откройте **Расширения → Apps Script**
2. Удалите весь код в редакторе и вставьте следующий скрипт:

```javascript
function doPost(e) {
  try {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = JSON.parse(e.postData.contents);

    sheet.appendRow([
      new Date().toLocaleString('ru-RU', { timeZone: 'Europe/Moscow' }),
      data.name || '',
      data.contact || '',
      data.company || '',
      data.project || '',
      '' // Статус обработки — заполняется вручную
    ]);

    return ContentService
      .createTextOutput(JSON.stringify({ status: 'ok' }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ status: 'error', message: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Сохраните скрипт (Ctrl+S / Cmd+S)

## Шаг 3. Развернуть как веб-приложение

1. Нажмите **Deploy → New deployment**
2. Нажмите на шестерёнку и выберите **Web app**
3. Настройки:
   - **Execute as:** Me (от моего имени)
   - **Who has access:** Anyone (любой)
4. Нажмите **Deploy**
5. При первом развёртывании Google попросит авторизацию — нажмите **Authorize access**, выберите ваш аккаунт, нажмите **Advanced → Go to ... (unsafe)** → **Allow**
6. Скопируйте **URL** веб-приложения (выглядит как `https://script.google.com/macros/s/.../exec`)

> **Важно:** параметр «Anyone» в настройках скрипта **НЕ** открывает доступ к вашей таблице. Это только разрешение на отправку данных через форму на сайте. Таблица остаётся полностью приватной — видеть и редактировать её можете только вы.

## Шаг 4. Вставить URL в код сайта

1. Откройте файл `index.html`
2. Найдите строку:
   ```
   const GOOGLE_SCRIPT_URL = 'ВСТАВЬТЕ_URL_GOOGLE_APPS_SCRIPT';
   ```
3. Замените `ВСТАВЬТЕ_URL_GOOGLE_APPS_SCRIPT` на скопированный URL скрипта
4. Сохраните файл

## Готово

Теперь при отправке формы на сайте данные будут автоматически записываться в вашу Google-таблицу. Столбец «Статус обработки» можно заполнять вручную для отслеживания работы с заявками.

## Если нужно обновить скрипт

При изменении скрипта нужно создать **новое развёртывание**: Deploy → New deployment. Старый URL перестанет работать — не забудьте обновить его в `index.html`.
