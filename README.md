# ISL Live: speech to Indian Sign Language, in real time

A classroom prototype: the teacher speaks, the screen shows a signer (stitched
ISL video clips), live captions, and the sign order. Runs fully offline on one laptop.

```
 Mic (laptop or USB wireless receiver)
   │  picked in the browser dropdown
   ▼
 Browser: capture → resample to 16 kHz PCM ──WebSocket──▶ Python server
                                                            │ Vosk (offline speech-to-text, Indian English)
                                                            │ partial text → live caption
                                                            │ final sentence → ISL gloss rules (spaCy)
                                                            │ gloss → clip lookup / fingerspelling
   ◀────────────── list of clips ──────────────────────────┘
 Browser: two <video> players alternate and preload the next clip,
          so clips play back to back without gaps
```

Example: "Tomorrow we will go to the zoo" → `TOMORROW WE ZOO GO`
"What is your name" → `YOUR NAME WHAT` · "I don't understand" → `I UNDERSTAND NOT`

## Setup (Windows, macOS or Linux)

1. Install Python 3.10+ and Chrome or Edge.
2. In this folder:
   ```
   pip install -r requirements.txt
   python -m spacy download en_core_web_sm
   ```
3. Download an offline speech model from https://alphacephei.com/vosk/models
   and unzip it into `models/`:
   * `vosk-model-small-en-in-0.4` (about 36 MB, Indian English, fast; start here)
   * `vosk-model-en-in-0.5` (about 1 GB, more accurate; needs a stronger laptop)

   The server picks the first `models/vosk-model*` folder, or set `VOSK_MODEL=path`.
4. Run `python server.py` (or double-click `run.bat`) and open http://localhost:8000

No model yet? The app falls back to Chrome’s built-in speech recognition
(needs internet, and always uses the system default mic). You can also type
sentences in the side panel to test without a mic.

## Choosing the microphone

The Microphone dropdown lists every input, including a USB wireless receiver.
Plug it in and it appears automatically; switching while listening restarts
the stream on the new device. The level bar shows whether audio is arriving.

## 3D signer with live speech

Open **http://localhost:8000/avatar** (or click “3D signer” in the app). Press
**Start listening** and teach normally. Every word that has a sign (sun, water,
evaporates, clouds, rain…) is highlighted in the caption and signed by the 3D
signer. If you talk faster than it can sign, words wait in the “Up next” queue,
and it signs a little faster until it catches up.

* The **Live** tab lists every trigger word. Add your own as `word = SIGN`.
* The **Edit signs** tab lets an ISL teacher fix or create signs. New signs
  become trigger words automatically.
* Signs and words are saved in this browser on this laptop.

## Adding sign clips

See `clips/README.md`. Short version: `clips/HELLO.mp4`, `clips/GOOD_MORNING.mp4`,
`clips/A.mp4`… Words without a clip are fingerspelled from the letter clips;
if letter clips are missing too, the word appears as a large text card.

Where clips can come from:
* **Record your own (recommended).** One ISL teacher, one background, one camera
  position. Start with ~150–300 classroom words (greetings, instructions, subjects,
  numbers, days, school objects). This also matches the signs your students actually use.
* **ISLRTC** (Indian Sign Language Research and Training Centre) publishes an ISL
  dictionary of several thousand signs. Ask them for permission before reusing videos.
* Research datasets such as AI4Bharat’s INCLUDE check their licence first.

Run `python tools/normalize_clips.py clips_raw clips` to convert everything to the
same size, frame rate and H.264 format so playback stitches cleanly.

## Tuning the grammar

All rules live at the top of `isl_gloss.py` (words to drop, time words, tense
markers, phrases, synonyms). Run `python isl_gloss.py` to print test sentences.
**These rules are a starting point. Have an ISL teacher or Deaf adult review the
output before using it with students.** ISL also uses facial expression, space and
directional verbs that word-by-word clips cannot fully capture.

## Known limits of this prototype

* English only. Vosk also has Hindi models; Hindi/Kannada need their own gloss rules.
* Speech arrives as a sentence after the speaker pauses, so signing lags speech by
  1–3 s. "Speed up when behind" plays faster when the queue builds up.
* Accuracy drops with classroom noise; a lapel or wireless mic helps a lot.
* Microphone access works on `localhost`. To open it from another device on the
  network you need HTTPS.

## Ideas for next steps

* Student-to-teacher direction: big tap buttons or a type box so deaf students can
  ask questions, shown or spoken aloud for the teacher.
* Topic vocabulary packs per subject (science, maths) and a teacher screen to
  record a missing sign in the moment.
* A 3D avatar (the ProDeaf / Hand Talk approach) driven by the same gloss, for
  words without recorded clips.
