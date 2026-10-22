# 🎓 AI Teacher — Voice AI Avatar (LiveKit + Synthesia)

An AI teacher you can talk to, live, from your browser. Students from **Class 1 to Class 12**, as well as **engineering students**, can simply speak their doubts and the teacher listens, thinks, and answers out loud through a lip-synced video avatar. No typing needed, just a full-screen voice-first experience.

---

## ✨ Features

- 🎙️ **Voice-only learning**: speak your question, hear the answer
- 🧑‍🏫 **Lip-synced AI avatar** powered by Synthesia Interactive Avatars
- 📚 **Built for all levels**: school students (Class 1–12) and engineering students
- 🇮🇳 **Hindi / Hinglish friendly**: Deepgram multi-language speech-to-text and Hindi text-to-speech
- 🔊 **Custom voice support** via Cartesia
- ⚡ **Real-time** conversation over WebRTC (LiveKit Cloud)
- 🖥️ **Simple frontend**: one plain HTML file, no build step
- 🔌 **Three interchangeable "brains"**:
  1. **Built-in LLM** (default): answers via LiveKit Inference, no extra LLM key needed
  2. **External HTTP service** (for example a Spring Boot API): the agent forwards each question to your own backend
  3. **Echo mode** (`FAKE_BRAIN=1`): repeats what you said, handy for testing the voice pipeline

---

## 🏗️ How It Works

```
Browser (index.html)  ──token──►  Flask token server (server.py)
        │                                   │
        └────────── WebRTC ─────────────────┘
                         │
                   LiveKit Cloud room
                         │
              Python agent (agent.py)
        ┌────────────────┼────────────────────┐
   Speech-to-text    LLM "brain"         Text-to-speech
 (Deepgram Nova-3)  (LiveKit Inference  (Cartesia, custom
                     or your own API)      voice supported)
                         │
                Synthesia interactive avatar
                (joins the room, publishes video)
```

| Part | File | Role |
| --- | --- | --- |
| Frontend | `index.html` | Full-screen avatar video, start button, mic mute and end-call controls |
| Token server | `server.py` | Flask app that serves the page and issues short-lived LiveKit tokens, so your secret never reaches the browser |
| Agent | `agent.py` | LiveKit Agents worker: speech-to-text, answer generation, text-to-speech, and driving the Synthesia avatar |

---

## 📋 Requirements

- Python 3.11+
- A [LiveKit Cloud](https://livekit.io) project (URL, API key, API secret)
- A [Synthesia](https://www.synthesia.io) workspace with Interactive Avatar access (API key and an **Interactive ID**)
- Optional: a [Cartesia](https://cartesia.ai) account for a custom voice

---

## 🚀 Setup

```bash
git clone https://github.com/Amiteshyadav3333/Teacher-.git
cd Teacher-

python3 -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt

cp .env.example .env.local
# open .env.local and fill in your own values
```

### Environment variables

All configuration lives in `.env.local` (git-ignored, never commit it). Use `.env.example` as a template.

| Variable | Required | Description |
| --- | --- | --- |
| `LIVEKIT_URL` | yes | LiveKit project URL, e.g. `wss://your-project.livekit.cloud` |
| `LIVEKIT_API_KEY` | yes | LiveKit API key |
| `LIVEKIT_API_SECRET` | yes | LiveKit API secret |
| `SYNTHESIA_API_KEY` | yes | Synthesia workspace API key |
| `SYNTHESIA_AVATAR_ID` | yes | Avatar **Interactive ID** (Avatars page → `•••` menu → *Copy Interactive ID*) |
| `CARTESIA_API_KEY` | no | Needed only for a custom Cartesia voice |
| `CARTESIA_VOICE_ID` | no | Your custom Cartesia voice ID |
| `TTS_MODEL` | no | LiveKit Inference TTS model, default `cartesia/sonic-3` |
| `TTS_VOICE` | no | Default voice ID when no custom voice is set |
| `TTS_LANGUAGE` | no | TTS language, default `hi` |
| `USE_SPRING` | no | Set to `1` to forward questions to an external HTTP service |
| `SPRING_AVATAR_URL` | no | That service's URL, default `http://127.0.0.1:8080/api/avatar/ask` |
| `FAKE_BRAIN` | no | Set to `1` for echo mode |
| `AGENT_NAME` | no | Only if you use explicit agent dispatch |

---

## ▶️ Run

Open two terminals, both with the virtual environment activated.

**Terminal 1: the agent**

```bash
python agent.py dev
```

**Terminal 2: the web server**

```bash
python server.py
```

Then open **http://localhost:8000**, click the start button, allow the microphone, and start asking your doubts.

> Use `localhost`, not your machine's IP address. Browsers only allow microphone access on `localhost` or HTTPS.

### Using an external HTTP brain

Set `USE_SPRING=1` and point `SPRING_AVATAR_URL` to your service. The agent sends:

```json
POST /api/avatar/ask
{ "question": "what the student said" }
```

and expects:

```json
{ "answer": "text the avatar should speak" }
```

### Testing without any brain

```bash
FAKE_BRAIN=1 python agent.py dev
```

The avatar will repeat what you said.

---

## 🎨 Customizing

- **Teaching style, subjects, and language:** edit `BUILTIN_INSTRUCTIONS` in `agent.py` (for example, make the teacher explain simply for young students or in more depth for engineering topics).
- **Voice:** set `CARTESIA_VOICE_ID` + `CARTESIA_API_KEY`, or change `TTS_VOICE`.
- **Look of the UI:** everything is in `index.html` (plain HTML, CSS, and JS).

---

## 📁 Project Structure

```
.
├── agent.py            # LiveKit agent: STT, answer generation, TTS, Synthesia avatar
├── server.py           # Flask server: serves the UI and issues LiveKit tokens
├── index.html          # Full-screen voice-only frontend
├── requirements.txt    # Python dependencies
├── .env.example        # Template for your secrets (copy to .env.local)
└── .gitignore
```

---

## 🛠️ Troubleshooting

| Symptom | Likely cause and fix |
| --- | --- |
| Page connects but no avatar appears | Check the agent terminal. `avatarIds does not exist or is not accessible` means the ID is not an Interactive ID for your workspace or plan. `a Synthesia API key is required` means `SYNTHESIA_API_KEY` is missing. |
| Avatar joins but stays silent | TTS problem. Custom Cartesia voices need `CARTESIA_API_KEY`; LiveKit Inference supports only Cartesia's default voices. |
| Avatar says it cannot generate an answer | External-brain mode only: your HTTP service is not running or the URL is wrong. |
| Microphone does not work | Open `http://localhost:8000`, not an IP address, and allow mic access. |
| `KeyError: 'LIVEKIT_URL'` on `server.py` | `.env.local` is missing or in the wrong folder. It must sit next to `server.py`. |

---

## 🔒 Security Notes

- Secrets are loaded from `.env.local`, which is git-ignored. **Never commit it.**
- If a key was ever committed or shared publicly, rotate it in the provider's dashboard.
- `server.py` runs Flask's development server. For public deployment, use a production WSGI server (such as gunicorn) behind HTTPS, and consider adding authentication to `/token`.

---

## 👨‍💻 Author

**Amitesh Yadav** — [@Amiteshyadav3333](https://github.com/Amiteshyadav3333)

---

⭐ If this project helps you, give it a star!
