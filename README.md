# 🎙️ Higgs Audio v3 TTS (4B) — Colab Notebook

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sruckh/HiggsAudioV3TTS/blob/main/higgs_audio_v3_tts_colab.ipynb)

A one-click Google Colab notebook that downloads, installs, and serves [**bosonai/higgs-audio-v3-tts-4b**](https://huggingface.co/bosonai/higgs-audio-v3-tts-4b) behind a public **Gradio** interface — for text-to-speech and zero-shot voice cloning.

> Click the **Open in Colab** badge above, switch to a GPU runtime, then run the cells top to bottom. The final cell prints a public `gradio.live` link.

---

## Features

- **End-to-end setup** — installs the [SGLang-Omni](https://github.com/sgl-project/sglang-omni) serving stack, downloads the ~5B-param model, and launches an OpenAI-compatible TTS server (`/v1/audio/speech`).
- **Public Gradio link** — `launch(share=True)` exposes a shareable `gradio.live` URL.
- **Text-to-Speech** — 100+ languages, with **playback and download** of generated audio.
- **Zero-shot voice cloning** — supply reference audio + its transcript to clone a voice.
- **Inline control tokens** — emotion, style, prosody, and sound-effect tags, selectable in the UI or written inline.

## Requirements

- A **GPU runtime** (`Runtime → Change runtime type → GPU`).
- The model is ~5B params (BF16, ≈10 GB weights). **L4 / A100 recommended**; a T4 (16 GB) may work but can be tight.
- A Hugging Face token is usually **not** required (public repo); paste one if a download fails with an auth error, or store it as a Colab secret named `HF_TOKEN`.

## Usage

1. Open the notebook in Colab via the badge above.
2. **Step 1** — verify the GPU.
3. **Step 2** — install SGLang-Omni + dependencies (a few minutes).
4. **Step 3** — download the model weights.
5. **Step 4** — launch the background TTS server and wait for it to become ready.
6. **Step 5** — launch the Gradio UI and open the printed public link.
7. **Step 6** — troubleshooting tips and a shutdown cell to free the GPU.

### Voice cloning

In the **Voice Cloning** tab, upload a short, clean reference clip and paste its **exact transcript** (this materially improves fidelity), then enter the text to speak in the cloned voice.

### Control tokens

Pick emotion / style / prosody from the dropdowns, or write tags inline in your text, e.g.:

```
<|emotion:amusement|><|prosody:expressive_high|>Wait, that was hilarious. <|sfx:laughter|>Hehe, I was not ready for that.
```

Delivery tokens (emotion / style / speed / pitch / expressiveness) shape the whole turn and go at the start. Positional tokens (`<|prosody:pause|>`, `<|sfx:…|>`) go inline where they fire; pair every `<|sfx:…|>` with its onomatopoeia.

## How it works

```
Gradio UI  ──HTTP POST /v1/audio/speech──▶  sgl-omni server  ──▶  higgs-audio-v3-tts-4b
 (share=True)                               (localhost:8000)        (GPU, 24 kHz WAV out)
```

The notebook runs the SGLang-Omni server as a background process inside the Colab VM; the Gradio app calls its local OpenAI-compatible endpoint and returns WAV audio for playback/download.

## License

The **model** is released under the *Boson Higgs Audio v3 Research and Non-Commercial License*. Voice cloning without consent, impersonation, fraud, or any unlawful use is prohibited; commercial / hosted use requires a separate commercial license from Boson AI. See the [model card](https://huggingface.co/bosonai/higgs-audio-v3-tts-4b) for details.
