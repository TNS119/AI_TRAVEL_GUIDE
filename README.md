# AI Travel Guide

AI Travel Guide is a small web app that creates spoken travel guides for selected
destinations. The browser frontend lets you choose a destination, guide length,
language, and voice. A Flask backend asks Google Gemini to write the guide, sends
the text to Murf for speech generation, and returns the description and MP3 audio
as Base64.

## Features

- Destination cards for selected places in India.
- Summary and Detailed guide styles.
- Guide generation in English, Hindi, Tamil, and Telugu.
- Male and female voice choices for each supported language.
- An on-page transcript and audio player.
- A Flask JSON API with CORS enabled for local frontend development.

## Project structure

```text
AI_TRAVEL_GUIDE/
├── Backend/
│   ├── app.py
│   ├── example.env
│   └── requirements.txt
├── Frontend/
│   ├── index.html
│   └── index.js
├── .gitignore
└── README.md
```

## Requirements

- Python 3.10 or newer.
- A Google Gemini API key.
- A Murf API key.
- A modern browser and an internet connection. The frontend loads Tailwind CSS
  and fonts from CDNs, while the backend calls the Gemini and Murf APIs.

## Configuration

1. Copy `Backend/example.env` to `Backend/.env`.
2. Add your own API keys to `Backend/.env`:

   ```dotenv
   GEMINI_API_KEY=your_gemini_api_key
   MURF_API_KEY=your_murf_api_key
   ```

Keep real credentials private. `.env` files are ignored by Git; do not put
working API keys in `example.env` or commit them.

## Install and run on Windows

Open PowerShell in the project root:

```powershell
cd Backend
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
Copy-Item example.env .env
```

Edit `Backend/.env` with your API keys, then start the backend:

```powershell
.\.venv\Scripts\python.exe app.py
```

The API runs at `http://127.0.0.1:5000`.

In a second terminal, serve the frontend. For example, if Node.js is installed,
run from the project root:

```powershell
npx --yes http-server .\Frontend -p 8080
```

Then open `http://127.0.0.1:8080` in your browser. Alternatively, open
`Frontend/index.html` with VS Code Live Server.

## Install and run on macOS or Linux

From the project root:

```bash
cd Backend
python3 -m venv .venv
./.venv/bin/python -m pip install -r requirements.txt
cp example.env .env
```

Edit `Backend/.env` with your API keys, then start the backend:

```bash
./.venv/bin/python app.py
```

In a second terminal, serve the frontend with Node.js:

```bash
npx --yes http-server ./Frontend -p 8080
```

Open `http://127.0.0.1:8080` in your browser.

## Using the app

1. Start the Flask backend and frontend server.
2. Open the frontend in your browser.
3. Select one of the destination cards.
4. Choose Summary or Detailed, a language, and a voice.
5. Select **Generate Audio Guide**.
6. Read the transcript or play the generated audio.

The frontend currently sends requests to
`http://127.0.0.1:5000/generate-audio-guide`. If the backend runs on a different
host or port, update `GENERATE_AUDIO_GUIDE_API_URL` in `Frontend/index.js`.

## API

### `POST /generate-audio-guide`

The endpoint expects a JSON request body:

```json
{
  "place": "Taj Mahal",
  "answerType": "Summary",
  "language": "English",
  "voiceId": "Matthew",
  "locale": "en-US"
}
```

Supported values used by the frontend:

| Field | Values |
| --- | --- |
| `answerType` | `Summary`, `Detailed` |
| `language` | `English`, `Hindi`, `Tamil`, `Telugu` |
| `voiceId` | Selected by the frontend for the chosen language and voice |
| `locale` | `en-US`, `hi-IN`, `ta-IN`, or `te-IN` |

On success, the backend returns JSON in this shape:

```json
{
  "description": "Generated travel guide text...",
  "audioBase64": "..."
}
```

The frontend uses `audioBase64` to create a playable MP3 data URL. The request
requires valid Gemini and Murf API keys and network access to both services.

## Troubleshooting

- **Dependency installation fails**: check the Render build logs and verify
  `Backend/requirements.txt` installs successfully in a local virtual environment.
- **Missing or invalid API key errors**: check that `Backend/.env` contains
  `GEMINI_API_KEY` and `MURF_API_KEY`, with valid values and no surrounding
  quotes unless required by your key.
- **Browser connection or CORS errors**: verify the Flask backend is running on
  port `5000` and that the frontend API URL matches it. Check the backend
  terminal for exceptions; a server error can appear as a failed browser
  request.
- **Audio generation fails**: check backend output and confirm the Murf request
  accepts the selected voice, locale, and generated text for your account.
- **Gemini generation fails**: check backend output, API access, and the model
  configured in `Backend/app.py`.

## Deploy to Render

The backend and frontend can be deployed as separate Render services. The
backend uses Gunicorn in production; `app.run(debug=True)` is guarded for local
development and is not used by Gunicorn.

### Backend Web Service

Create a **Web Service** connected to the repository with:

| Setting | Value |
| --- | --- |
| Root Directory | `Backend` |
| Runtime | Python |
| Build Command | `pip install -r requirements.txt` |
| Start Command | `gunicorn --bind 0.0.0.0:$PORT --timeout 180 app:app` |

Add these environment variables in the Render service settings:

| Variable | Value |
| --- | --- |
| `GEMINI_API_KEY` | Your Gemini API key |
| `MURF_API_KEY` | Your Murf API key |

After the backend deploys, copy its Render URL, for example
`https://your-api.onrender.com`.

### Frontend Static Site

Create a **Static Site** from the same repository:

| Setting | Value |
| --- | --- |
| Build Command | Leave blank |
| Publish Directory | `Frontend` |

Before deploying the frontend, update `GENERATE_AUDIO_GUIDE_API_URL` in
`Frontend/index.js` to the backend URL:

```js
const GENERATE_AUDIO_GUIDE_API_URL =
  "https://your-api.onrender.com/generate-audio-guide";
```

Use the actual backend URL from Render.

### Deployment notes

- Do not commit `.env` files or put API keys in frontend JavaScript. The browser
  code is public; keep provider keys only in the backend environment.
- `gunicorn` and `requests` are listed in `Backend/requirements.txt`.
- Render provides the `PORT` environment variable; Gunicorn binds to it.
- This project uses synchronous Gemini and Murf API calls. Gunicorn's worker
  timeout is set to 180 seconds to allow time for generation.
- Flask debug mode is used only when running `python app.py` locally. Do not
  expose Flask's development server or enable debug mode in production.
