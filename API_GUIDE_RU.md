# Codex Web GPT API: полная инструкция

## Самое главное

Codex Web GPT уже установлен на Linux Mint и предоставляет API через существующую браузерную сессию ChatGPT.

- Официальный OpenAI API key не нужен.
- Отдельная оплата OpenAI API не используется.
- `CODEX_WEB_TOKEN` — это только пароль от твоего API-моста.
- API доступен через защищённую сеть Tailscale.

Схема работы:

```text
Твой компьютер
      ↓
Codex Web GPT API на Linux-сервере
      ↓
Авторизованная браузерная ChatGPT-сессия
      ↓
Ответ ChatGPT
```

## Адреса и настройки

| Параметр | Значение |
|---|---|
| Пользователь сервера | `mg` |
| Публичный SSH-адрес | `24.150.94.103` |
| Tailscale-адрес | `100.82.137.49` |
| API-порт | `17842` |
| API Base URL | `http://100.82.137.49:17842/v1` |
| Версия приложения | `3.1.0` |

Для API всегда используй:

```text
http://100.82.137.49:17842/v1
```

Публичный адрес `24.150.94.103` предназначен для SSH. API-порт на публичном адресе не открыт.

## Что должно работать перед использованием

1. Linux-сервер включён и подключён к интернету.
2. Tailscale запущен на сервере и клиентском компьютере.
3. Оба устройства находятся в одной Tailscale-сети.
4. Пользователь `mg` вошёл в графическую сессию Linux Mint.
5. Codex Web GPT запущен.
6. В браузере Codex Web GPT выполнен вход в ChatGPT.

Codex Web GPT настроен на автоматический запуск после входа пользователя `mg` в Linux Mint.

## Проверка Tailscale

На клиентском компьютере выполни:

```sh
ping 100.82.137.49
```

Также можно проверить список устройств:

```sh
tailscale status
```

В списке должен быть сервер с адресом `100.82.137.49`.

Если сервер не отвечает:

- проверь, что сервер включён;
- проверь, что Tailscale запущен на обоих устройствах;
- проверь, что оба устройства используют одну Tailscale-сеть.

## Где находится CODEX_WEB_TOKEN

Токен хранится на сервере:

```text
/home/mg/.codex-chatgpt-web/secrets/external-api.token
```

Это не OpenAI API key. Токен защищает API-мост от посторонних пользователей.

### Посмотреть токен на сервере

Подключись к серверу:

```sh
ssh mg@100.82.137.49
```

Если подключение через Tailscale недоступно:

```sh
ssh mg@24.150.94.103
```

Покажи токен:

```sh
cat ~/.codex-chatgpt-web/secrets/external-api.token
```

Не публикуй полученную строку в GitHub, Telegram, документации или исходном коде.

## Использование API непосредственно на сервере

После подключения по SSH загрузи настройки:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"
```

Эти переменные действуют только в текущем окне терминала.

## Использование с другого Linux-компьютера или macOS

Скопируй токен с сервера:

```sh
scp mg@100.82.137.49:/home/mg/.codex-chatgpt-web/secrets/external-api.token ./codex-api.token
```

Ограничь доступ к файлу:

```sh
chmod 600 ./codex-api.token
```

Загрузи настройки:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(tr -d '\r\n' < ./codex-api.token)"
```

## Использование с Windows

Открой PowerShell и скопируй токен:

```powershell
scp mg@100.82.137.49:/home/mg/.codex-chatgpt-web/secrets/external-api.token "$HOME\codex-api.token"
```

Загрузи настройки:

```powershell
$env:CODEX_WEB_API_URL = "http://100.82.137.49:17842/v1"
$env:CODEX_WEB_TOKEN = (Get-Content "$HOME\codex-api.token" -Raw).Trim()
```

Переменные действуют до закрытия текущего окна PowerShell.

## Проверка доступности API

Сначала сделай запрос без токена:

```sh
curl -i http://100.82.137.49:17842/v1/models
```

Ожидаемый результат:

```text
HTTP/1.1 401 Unauthorized
```

`401 Unauthorized` означает, что сервер доступен и правильно требует токен.

## Получение списка моделей

### Linux или macOS

```sh
curl "$CODEX_WEB_API_URL/models" \
  -H "Authorization: Bearer $CODEX_WEB_TOKEN"
```

### Windows PowerShell

```powershell
Invoke-RestMethod `
  -Uri "$env:CODEX_WEB_API_URL/models" `
  -Headers @{
    Authorization = "Bearer $env:CODEX_WEB_TOKEN"
  }
```

Доступные модели:

| Модель | Назначение |
|---|---|
| `chatgpt-web/light` | Быстрые и простые задачи |
| `chatgpt-web/medium` | Обычные повседневные задачи |
| `chatgpt-web/high` | Более сложные задачи |
| `chatgpt-web/extra-high` | Максимальное рассуждение |
| `chatgpt-web/pro` | Режим ChatGPT Pro |

Для первого запроса используй `chatgpt-web/light`.

## Отправка первого запроса

### Linux или macOS

```sh
curl "$CODEX_WEB_API_URL/responses" \
  -H "Authorization: Bearer $CODEX_WEB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "chatgpt-web/light",
    "input": "Ответь одним словом: работает?"
  }'
```

### Windows PowerShell

```powershell
$body = @{
  model = "chatgpt-web/light"
  input = "Ответь одним словом: работает?"
} | ConvertTo-Json

Invoke-RestMethod `
  -Method Post `
  -Uri "$env:CODEX_WEB_API_URL/responses" `
  -Headers @{
    Authorization = "Bearer $env:CODEX_WEB_TOKEN"
  } `
  -ContentType "application/json" `
  -Body $body
```

Ответ придёт в формате JSON. Запрос может выполняться дольше обычного API, потому что ответ создаётся через браузерную ChatGPT-сессию.

## Пример на Python

Установи библиотеку:

```sh
python -m pip install requests
```

Создай файл `test_api.py`:

```python
import os
import requests

api_url = os.environ["CODEX_WEB_API_URL"]
token = os.environ["CODEX_WEB_TOKEN"]

response = requests.post(
    f"{api_url}/responses",
    headers={
        "Authorization": f"Bearer {token}",
        "Content-Type": "application/json",
    },
    json={
        "model": "chatgpt-web/light",
        "input": "Напиши короткое приветствие.",
    },
    timeout=300,
)

response.raise_for_status()
print(response.json())
```

Запусти:

```sh
python test_api.py
```

## Пример на Node.js

Нужен Node.js 18 или новее.

Создай файл `test-api.mjs`:

```javascript
const apiUrl = process.env.CODEX_WEB_API_URL;
const token = process.env.CODEX_WEB_TOKEN;

const response = await fetch(`${apiUrl}/responses`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "chatgpt-web/light",
    input: "Напиши короткое приветствие.",
  }),
});

if (!response.ok) {
  throw new Error(`API error: ${response.status} ${await response.text()}`);
}

console.log(await response.json());
```

Запусти:

```sh
node test-api.mjs
```

## Настройка сторонней программы

Если программа поддерживает собственный OpenAI-compatible Base URL и Responses API, укажи:

```text
Base URL: http://100.82.137.49:17842/v1
API token: содержимое external-api.token
Model: chatgpt-web/light
```

Если программа требует поле с названием `OpenAI API Key`, вставь туда `CODEX_WEB_TOKEN`. Это поле используется только для создания заголовка:

```http
Authorization: Bearer CODEX_WEB_TOKEN
```

Запрос всё равно выполняется через ChatGPT-сессию на сервере.

Сервер использует:

```text
GET /v1/models
POST /v1/responses
```

Программа, которая поддерживает только `/v1/chat/completions`, может не работать.

## Возможные ошибки

### `401 Unauthorized`

Причины:

- токен не передан;
- токен неправильный;
- в токен попал перенос строки.

Linux или macOS:

```sh
export CODEX_WEB_TOKEN="$(tr -d '\r\n' < ./codex-api.token)"
```

Windows PowerShell:

```powershell
$env:CODEX_WEB_TOKEN = (Get-Content "$HOME\codex-api.token" -Raw).Trim()
```

### Timeout или невозможно подключиться

Проверь:

```sh
ping 100.82.137.49
```

Убедись, что используешь адрес:

```text
http://100.82.137.49:17842/v1
```

### `404 Not Found`

Проверь наличие `/v1` в адресе.

Правильно:

```text
http://100.82.137.49:17842/v1/responses
```

Неправильно:

```text
http://100.82.137.49:17842/responses
```

### `429 Too Many Requests`

ChatGPT временно ограничил количество запросов. Подожди и повтори запрос либо используй `chatgpt-web/light`.

### API доступен, но ChatGPT не отвечает

1. Войди в графическую Linux Mint-сессию.
2. Открой Codex Web GPT.
3. Проверь, что ChatGPT авторизован.
4. Если ChatGPT просит войти, выполни вход вручную.
5. Повтори запрос.

## Проверка сервера

Подключись по SSH:

```sh
ssh mg@100.82.137.49
```

Проверь порт:

```sh
ss -ltn | grep 17842
```

Ожидаемый адрес:

```text
100.82.137.49:17842
```

Проверь API:

```sh
TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"

curl http://100.82.137.49:17842/v1/models \
  -H "Authorization: Bearer $TOKEN"
```

## Ручной запуск Codex Web GPT

Если приложение не запустилось автоматически, выполни команду внутри графической Linux Mint-сессии:

```sh
"/home/mg/.local/lib/codex-web-gpt/3.1.0/Codex Web GPT.AppImage"
```

Автозапуск настроен в файле:

```text
/home/mg/.config/autostart/codex-web-gpt.desktop
```

## Безопасность

Нельзя:

- публиковать токен на GitHub;
- вставлять токен в исходный код;
- отправлять токен другим людям;
- передавать токен внутри URL;
- открывать API-порт напрямую в интернет без HTTPS и дополнительной защиты.

Правильно:

```http
Authorization: Bearer TOKEN
```

Неправильно:

```text
http://server/v1/responses?token=TOKEN
```

## Короткая памятка

На сервере достаточно выполнить:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"

curl "$CODEX_WEB_API_URL/responses" \
  -H "Authorization: Bearer $CODEX_WEB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "chatgpt-web/light",
    "input": "Привет!"
  }'
```

Главное: ответы создаёт текущая ChatGPT-сессия на сервере. `CODEX_WEB_TOKEN` только защищает доступ к API-мосту.
