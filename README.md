# Clue Buzzer

Single-file web app for trivia night. Tap **Listen** while the host reads the clue, and the page transcribes it, asks Claude (with live web search), and shows the three most likely responses with a confidence percentage for each. **Free run** keeps listening all night, grades itself from the host's revealed answers, and uses those answers to improve later clues. Dark, low-emission UI.

## How it works

- **Listening**: the browser's built-in speech recognizer (Web Speech API). Works in Chrome on Android/desktop and Safari on iOS 15+. The page must be served over `https` for the microphone to be allowed.
- **Answering**: a direct call from the browser to the Claude Messages API with the `web_search` server tool. Claude searches only when it is not confident, capped at two searches per clue, so most clues come back in a few seconds.
- **Confidence**: the model's own probability estimate per response, descending, summing to at most 100%.
- **Single clue mode**: after the host stops talking for 1.5 s (adjustable) the clue is submitted automatically. You can also tap **Answer**, or edit the transcript and press Enter.
- Screen wake lock keeps the display on while listening.

## Free run

Tap **Free run** once and leave the phone on the table.

1. Every pause-separated utterance is sent to Haiku 5.5, which labels it as a new clue, the rest of the current clue, a revealed answer, or chatter.
2. A clue is answered immediately with your chosen model (quick pass, up to 2 searches).
3. **Self-check**: if the top confidence is below the threshold (default 90%), a second pass runs with up to 4 searches and medium effort, and the card is marked *verified* (the status line says whether the answer changed).
4. **Grading**: when the host reads the correct answers, the classifier matches them to pending clues and records whether our #1 was right, a lower guess was right, or we missed. The game log shows a running score. You can also type the host's answer on any pending entry.
5. **Learning**: graded clues and short lessons (for example "host wants full names") are added to every later prompt, including the verification pass, so Claude infers the round's theme and the host's answer style and writes better search queries. Memory persists in the browser until you tap **New game**.

## Setup

1. Get an Anthropic API key at https://platform.claude.com/ (Settings, API keys).
2. Open the hosted copy: https://billstewart196922-dot.github.io/clue-buzzer/ (GitHub Pages from this repo). To self-host instead, put `index.html` on any https host.
3. Open the page on your phone, expand **Settings**, paste the key. It is stored only in that browser's local storage.
4. Allow microphone access when prompted. Add the page to the home screen for a full-screen app.

## Settings

| Setting | Default | Notes |
|---|---|---|
| Model for answers | Opus 5.5 | Sonnet 5.5 and Haiku 5.5 are faster and cheaper. Free run always classifies with Haiku 5.5. |
| Search the web when unsure | on | Off = knowledge only, fastest. Also disables the verification pass. |
| Auto-answer after silence | on | Single clue mode only. |
| Silence that ends an utterance | 1.5 s | Raise it if the host pauses mid-clue. |
| Re-check below (%) | 90 | Free run verification threshold. 0 disables, 100 verifies everything. |
| Phrase responses as questions | on | "Who is…?" vs the bare answer. |

## Cost

Per clue: one answer request (~1k output tokens) plus $0.01 per web search when one runs. Free run adds one tiny Haiku call per utterance and an occasional verification pass with up to 4 searches. A full night is a dollar or two on Sonnet, several dollars on Opus.

## Limits

- Opened inside claude.ai as an artifact, the page runs in a locked frame: no microphone, no Free run, no web search. It answers typed clues from Claude's own knowledge on your account, and still accepts typed correct answers for the memory.
- Speech recognition quality depends on the room. Keep the phone near a speaker and correct the transcript if needed before answering.
- The classifier is doing real work in a noisy environment: expect an occasional contestant guess to be taken as a clue. Those entries are harmless and can be ignored.
