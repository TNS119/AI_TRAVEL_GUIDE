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
.\.venv\Scripts\python.exe -m pip install -r requirements.txt requests
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
./.venv/bin/python -m pip install -r requirements.txt requests
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

### Dependency note

`app.py` imports the `requests` package, but `requests` is not currently listed
in `Backend/requirements.txt`. The install commands above include it explicitly
so the backend can run. If you maintain the dependency list, add `requests` to
that file and then `pip install -r requirements.txt` will be sufficient.

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

- **`ModuleNotFoundError: No module named 'requests'`**: install it with
  `python -m pip install requests`, or add it to `Backend/requirements.txt`.
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

## Security and deployment

The Flask development server is configured with `debug=True` and is intended
for local development. Do not expose it directly to the public internet. For
deployment, use a production WSGI server, disable debug mode, configure
environment variables through the host, restrict CORS to the frontend origin,
and update the frontend API URL to the deployed backend.
