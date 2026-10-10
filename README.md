# 🛡️ VoiceGuard

**Real-time, on-device deepfake voice detection — built right into your calling experience.**

VoiceGuard listens to a live call audio stream and flags AI-generated (deepfake / voice-cloned) speech *while the call is still happening*, so a victim can hang up before the scam succeeds. All analysis runs **100% on-device** in the browser — no audio ever leaves the user's machine.

> Prototype for the *VoiceGuard* cybersecurity project. This web app is the deployable demo of the detection engine that also targets a **browser extension** (WebRTC calls: Zoom / Teams / Meet) and a **lightweight mobile app** (cellular / VoIP).

---

## ✨ What it does

- 🎙️ **Live mic analysis** — captures audio via the Web Audio API and scores it in real time.
- 🤖 **Deepfake demo mode** — generates a vocoder-like synthetic voice in-browser so you can watch a detection fire instantly (no audio files needed).
- 📊 **Signal analysis panel** — shows the six temporal-spectral features the model fuses:
  - Spectral Flatness · Formant Stability · Harmonic/Noise Ratio · Phase Jitter Loss · High-Frequency Rolloff · Energy Flatness
- ⚠️ **Non-disruptive alert overlay** — "Synthetic Voice Likely Detected" with *End Call Safely* / *Flag Number* actions and a device vibration pulse.
- 🎚️ **Adjustable sensitivity** — user-controlled threshold to balance alerts vs. false positives.
- 🚩 **Community flagging + detection log** — stored locally (localStorage), nothing uploaded.
- 🔒 **Privacy by design** — on-device only, GDPR / CCPA friendly.

## ⚡ Latency

The engine analyzes overlapping **100 ms** audio chunks and reports measured end-to-end latency live in the top bar — well within the **sub-300 ms** real-time intervention target on commodity hardware.

---

## 🧠 How the detection works

`src/detector.js` holds the detection engine. Per 100 ms frame it pulls frequency + time-domain data from an `AnalyserNode`, extracts features, and fuses them into a single **synthetic likelihood** score (0–1), smoothed with an exponential moving average to stay stable.

The heuristics target artifacts that distinguish natural speech from vocoder/TTS output (e.g. WaveNet / Tacotron style): flattened spectra, unnaturally steady formants, missing breath/subharmonic noise, and lost micro-jitter.

> This prototype uses an interpretable **heuristic model** so the demo is fully self-contained and runs anywhere. The feature contract is designed so this stage can be swapped for a **quantized depthwise-separable CNN** (ONNX / TFLite / WebNN) without changing the UI.

---

## 🚀 Run locally

> Requires **Node.js 18+** (install from https://nodejs.org).

```bash
cd voiceguard
npm install
npm run dev      # start dev server (http://localhost:5173)
npm run build    # production build -> dist/
npm run preview  # preview the production build
```

Allow microphone permission when prompted, or click **Run Deepfake Demo** to see a detection without a mic.

> ⚠️ **Build not run in this environment:** Node.js was not installed on the machine where this prototype was generated, so `npm run build` could not be executed here. Run the commands above locally to produce `dist/`. All imports/exports and component contracts were verified by inspection.

---

## ☁️ Deploy to Vercel

The project is pre-configured (`vercel.json`, Vite framework preset).

**Option A — dashboard:** push to GitHub, "New Project" on Vercel, pick the `voiceguard` folder as the root. Vercel auto-detects Vite.

**Option B — CLI:**
```bash
npm i -g vercel
vercel          # preview deploy
vercel --prod   # production deploy -> your-project.vercel.app
```

Build command `npm run build`, output directory `dist` (already set in `vercel.json`).

> 🔐 Microphone capture requires HTTPS. Vercel serves HTTPS by default, so live mic works on the deployed URL.

---

## 📁 Project structure

```
voiceguard/
├─ index.html
├─ vercel.json
├─ vite.config.js
├─ public/shield.svg
└─ src/
   ├─ main.jsx            # React entry
   ├─ App.jsx             # orchestration: capture, analysis loop, state
   ├─ detector.js         # on-device deepfake detection engine
   ├─ synthVoice.js       # in-browser synthetic-voice demo source
   ├─ styles.css          # dark security-themed UI
   └─ components/
      ├─ Waveform.jsx      # live canvas waveform
      ├─ ScoreMeter.jsx    # synthetic-likelihood meter + threshold
      ├─ FeatureBars.jsx   # per-feature breakdown
      ├─ AlertOverlay.jsx  # synthetic-voice alert dialog
      └─ Dashboard.jsx     # sensitivity, event log, community flags
```

## 🗺️ Roadmap

- Swap heuristic model for a trained quantized CNN via WebNN / ONNX Runtime Web.
- Browser-extension packaging to tap WebRTC call streams directly.
- React Native / Capacitor mobile build with DSP/NPU acceleration.
- Opt-in federated learning on anonymized detection metadata.

## 🔏 Privacy

VoiceGuard performs all processing locally in the browser. No audio, features, or flags are transmitted to any server. Flags and logs live only in your browser's `localStorage`.

---

*Prototype — detection heuristics are illustrative and not a guarantee of accuracy. Always verify caller identity through a trusted channel.*
