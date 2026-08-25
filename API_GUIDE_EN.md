# Codex Web GPT API: Complete Guide

## The most important information

Codex Web GPT is already installed on the Linux Mint server. It provides an API backed by the existing browser-based ChatGPT session.

- You do not need an official OpenAI API key.
- This setup does not use separately billed OpenAI API access.
- `CODEX_WEB_TOKEN` is only the password protecting your API bridge.
- The API is available through the private Tailscale network.

How it works:

```text
Your computer
      ↓
Codex Web GPT API on the Linux server
      ↓
Authenticated browser-based ChatGPT session
      ↓
ChatGPT response
```

## Server addresses and settings

| Setting | Value |
|---|---|
| Server user | `mg` |
| Public SSH address | `24.150.94.103` |
| Tailscale address | `100.82.137.49` |
| API port | `17842` |
| API base URL | `http://100.82.137.49:17842/v1` |
| Application version | `3.1.0` |

Always use this address for the API:

```text
http://100.82.137.49:17842/v1
```

The public address `24.150.94.103` is used for SSH. The API port is not exposed on the public address.

## What must be running

Before using the API, make sure that:

1. The Linux server is powered on and connected to the internet.
2. Tailscale is running on the server and on your client computer.
3. Both devices belong to the same Tailscale network.
4. User `mg` is signed in to the Linux Mint desktop session.
5. Codex Web GPT is running.
6. ChatGPT is signed in inside the Codex Web GPT browser.

Codex Web GPT is configured to start automatically after user `mg` signs in to the Linux Mint desktop.

## Check the Tailscale connection

Run this command on your client computer:

```sh
ping 100.82.137.49
```

You can also list your Tailscale devices:

```sh
tailscale status
```

The list should contain the server with address `100.82.137.49`.

If the server does not respond:

- make sure the server is powered on;
- make sure Tailscale is running on both devices;
- make sure both devices belong to the same Tailscale network.

## Where to find CODEX_WEB_TOKEN

The token is stored on the server at:

```text
/home/mg/.codex-chatgpt-web/secrets/external-api.token
```

This is not an OpenAI API key. It protects the API bridge from unauthorized access.

### View the token on the server

Connect to the server through Tailscale:

```sh
ssh mg@100.82.137.49
```

If the Tailscale SSH route is unavailable, use the public SSH address:

```sh
ssh mg@24.150.94.103
```

Display the token:

```sh
cat ~/.codex-chatgpt-web/secrets/external-api.token
```

Do not publish the resulting value in GitHub, Telegram, documentation, or source code.

## Use the API directly on the server

After connecting over SSH, load the settings into the current terminal:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"
```

These environment variables remain available only in the current terminal session.

## Use the API from another Linux computer or macOS

Copy the token from the server:

```sh
scp mg@100.82.137.49:/home/mg/.codex-chatgpt-web/secrets/external-api.token ./codex-api.token
```

Restrict access to the copied file:

```sh
chmod 600 ./codex-api.token
```

Load the settings:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(tr -d '\r\n' < ./codex-api.token)"
```

## Use the API from Windows

Open PowerShell and copy the token:

```powershell
scp mg@100.82.137.49:/home/mg/.codex-chatgpt-web/secrets/external-api.token "$HOME\codex-api.token"
```

Load the settings:

```powershell
$env:CODEX_WEB_API_URL = "http://100.82.137.49:17842/v1"
$env:CODEX_WEB_TOKEN = (Get-Content "$HOME\codex-api.token" -Raw).Trim()
```

These variables remain available until you close the current PowerShell window.

## Check whether the API is reachable

First, send a request without a token:

```sh
curl -i http://100.82.137.49:17842/v1/models
```

Expected result:

```text
HTTP/1.1 401 Unauthorized
```

`401 Unauthorized` is a good result in this test. It means the server is reachable and correctly requires authentication.

## Get the available models

### Linux or macOS

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

Available models:

| Model | Intended use |
|---|---|
| `chatgpt-web/light` | Fast and simple tasks |
| `chatgpt-web/medium` | Normal everyday tasks |
| `chatgpt-web/high` | More difficult tasks |
| `chatgpt-web/extra-high` | Maximum reasoning effort |
| `chatgpt-web/pro` | ChatGPT Pro mode |

Use `chatgpt-web/light` for your first request.

## Send your first request

### Linux or macOS

```sh
curl "$CODEX_WEB_API_URL/responses" \
  -H "Authorization: Bearer $CODEX_WEB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "chatgpt-web/light",
    "input": "Reply with one word: working?"
  }'
```

### Windows PowerShell

```powershell
$body = @{
  model = "chatgpt-web/light"
  input = "Reply with one word: working?"
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

The response is returned as JSON. A request can take longer than a normal API request because the answer is generated through the browser-based ChatGPT session.

## Python example

Install the required library:

```sh
python -m pip install requests
```

Create a file named `test_api.py`:

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
        "input": "Write a short greeting.",
    },
    timeout=300,
)

response.raise_for_status()
print(response.json())
```

Run it:

```sh
python test_api.py
```

## Node.js example

Node.js 18 or newer is required.

Create a file named `test-api.mjs`:

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
    input: "Write a short greeting.",
  }),
});

if (!response.ok) {
  throw new Error(`API error: ${response.status} ${await response.text()}`);
}

console.log(await response.json());
```

Run it:

```sh
node test-api.mjs
```

## Configure a third-party application

If an application supports a custom OpenAI-compatible base URL and the Responses API, use these values:

```text
Base URL: http://100.82.137.49:17842/v1
API token: contents of external-api.token
Model: chatgpt-web/light
```

If the application requires a field named `OpenAI API Key`, enter `CODEX_WEB_TOKEN` in that field. The application uses it only to create this HTTP header:

```http
Authorization: Bearer CODEX_WEB_TOKEN
```

The actual request is still handled through the ChatGPT session on your server.

The server provides these endpoints:

```text
GET /v1/models
POST /v1/responses
```

An application that supports only `/v1/chat/completions` may not work with this server.

## Troubleshooting

### `401 Unauthorized`

Possible causes:

- the token was not included;
- the token is incorrect;
- the token contains an unwanted newline.

Linux or macOS:

```sh
export CODEX_WEB_TOKEN="$(tr -d '\r\n' < ./codex-api.token)"
```

Windows PowerShell:

```powershell
$env:CODEX_WEB_TOKEN = (Get-Content "$HOME\codex-api.token" -Raw).Trim()
```

### Connection timeout or connection refused

Check the Tailscale connection:

```sh
ping 100.82.137.49
```

Make sure you are using:

```text
http://100.82.137.49:17842/v1
```

### `404 Not Found`

Make sure `/v1` is included in the address.

Correct:

```text
http://100.82.137.49:17842/v1/responses
```

Incorrect:

```text
http://100.82.137.49:17842/responses
```

### `429 Too Many Requests`

ChatGPT has temporarily limited the number of requests. Wait and try again, or select `chatgpt-web/light`.

### The API is reachable, but ChatGPT does not respond

1. Sign in to the Linux Mint desktop session.
2. Open Codex Web GPT.
3. Confirm that ChatGPT is signed in.
4. If ChatGPT asks you to sign in, complete the login manually.
5. Send the API request again.

## Check the server

Connect over SSH:

```sh
ssh mg@100.82.137.49
```

Check the listening port:

```sh
ss -ltn | grep 17842
```

Expected address:

```text
100.82.137.49:17842
```

Test the API from the server:

```sh
TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"

curl http://100.82.137.49:17842/v1/models \
  -H "Authorization: Bearer $TOKEN"
```

## Start Codex Web GPT manually

If the application did not start automatically, run this command inside the graphical Linux Mint session:

```sh
"/home/mg/.local/lib/codex-web-gpt/3.1.0/Codex Web GPT.AppImage"
```

The autostart configuration is stored at:

```text
/home/mg/.config/autostart/codex-web-gpt.desktop
```

## Security rules

Do not:

- publish the token on GitHub;
- place the token directly in source code;
- send the token to other people;
- include the token in a URL;
- expose the API port directly to the internet without HTTPS and additional protection.

Correct:

```http
Authorization: Bearer TOKEN
```

Incorrect:

```text
http://server/v1/responses?token=TOKEN
```

## Quick reference

On the server, these commands are enough:

```sh
export CODEX_WEB_API_URL="http://100.82.137.49:17842/v1"
export CODEX_WEB_TOKEN="$(cat ~/.codex-chatgpt-web/secrets/external-api.token)"

curl "$CODEX_WEB_API_URL/responses" \
  -H "Authorization: Bearer $CODEX_WEB_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "chatgpt-web/light",
    "input": "Hello!"
  }'
```

The important point: the current ChatGPT session on the server generates the responses. `CODEX_WEB_TOKEN` only protects access to the API bridge.
