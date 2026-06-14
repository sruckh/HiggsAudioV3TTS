# AGENTS.md — HiggsAudioV3TTS

Project-specific guidance for coding agents working on the Higgs Audio v3 TTS (4B) Colab notebook.

## Project at a glance

- **Deliverable**: a single Google Colab notebook, `higgs_audio_v3_tts_colab.ipynb`, plus this README/AGENTS metadata.
- **Purpose**: one-click install, serve, and UI for [`bosonai/higgs-audio-v3-tts-4b`](https://huggingface.co/bosonai/higgs-audio-v3-tts-4b) text-to-speech and zero-shot voice cloning.
- **Runtime architecture**:
  ```
  Gradio UI (share=True → gradio.live link)
    → HTTP POST /v1/audio/speech
    → sgl-omni server on localhost:8000
    → Higgs Audio v3 TTS 4B model on GPU
    → 24 kHz WAV
  ```

## Hard requirements

- **GPU**: Ampere-or-newer, compute capability **sm_80+** (A100, L4). Free-tier Colab Tesla T4 (sm_75) is **not supported** — sglang's custom CUDA kernels ship only for sm_80+. Cell 1 raises a clear `SystemExit` on unsupported GPUs.
- **VRAM**: ~10 GB weights (BF16, ~5B params). ≥16 GB workable; 24 GB+ comfortable.
- **Environment**: notebook is written for Google Colab (`/content/` paths). Running locally requires adapting Colab assumptions.

## Tech stack

- Python 3, Jupyter notebook
- `torch==2.9.1`, `torchvision==0.24.1`, `torchaudio==2.9.1`
- **Serving**: [sgl-project/sglang-omni](https://github.com/sgl-project/sglang-omni), cloned to `/content/sglang-omni`
- **Package installer**: `uv` (not plain pip) — required because `sglang-omni` uses uv's `override-dependencies` to resolve a protobuf conflict
- **UI**: `gradio>=4.44`, `requests`, `soundfile`, `huggingface_hub`
- **Optional**: matching `flash-attn` wheel on sm_80+; otherwise sglang uses another attention backend

## How to run

1. Open `higgs_audio_v3_tts_colab.ipynb` in Colab via the badge in README.md.
2. Use a supported GPU runtime (L4 / A100).
3. Run cells sequentially:
   1. GPU check
   2. Install SGLang-Omni and dependencies
   3. (Optional) Hugging Face token & download model
   4. Launch TTS server
   5. Launch Gradio interface (public link)
   6. Troubleshooting & shutdown

There is no local `python main.py`, test suite, or build script.

## Critical conventions and caveats

- **HF token**: optional. Code tries the `HF_TOKEN` cell parameter, then the Colab secret `HF_TOKEN` via `google.colab.userdata`, then the env var. The model is public, so a token is usually unnecessary.
- **Serve by repo id**: pass `--model-path bosonai/higgs-audio-v3-tts-4b`, **not** the full Hugging Face cache path. sglang derives a ZMQ IPC socket name from the path string; the full cache path overflows the 107-character `AF_UNIX sun_path` limit.
- **Uninstall TensorFlow before serving**: Colab preloads TF, whose bundled C++ protobuf runtime conflicts with sglang/grpc and causes a SIGABRT with `"File already exists in database: google/protobuf/duration.proto"`. The serve cell uninstalls TF automatically.
- **Protobuf pure-Python fallback**: the serve process sets `PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=python` as an extra safeguard.
- **Control tokens** (model-card vocabularies):
  - Delivery tokens (prefix): `<|emotion:{name}|>`, `<|style:{name}|>`, `<|prosody:{speed|pitch|expressiveness}|>`
  - Inline/positional: `<|prosody:pause|>`, `<|sfx:{name}|>` — always pair `<|sfx:…|>` with its onomatopoeia.
- **Voice cloning**: upload a short, clean reference clip and paste its **exact transcript** in the reference text field for best fidelity.
- **`.kilo/`**: Kilo Code plugin configuration directory; gitignored and not part of the runtime.

## Search and exploration tools

When investigating or modifying the project, prefer these indexes in this order:

1. **Serena memories** — read `mem:core` first; it references `mem:tech_stack`, `mem:conventions`, `mem:suggested_commands`, and `mem:task_completion`.
2. **jdocmunch** — documentation/notebook index: `local/HiggsAudioV3TTS-docs`. Good for natural-language and code-block searches inside the notebook and README.
3. **jcodemunch** — `sruckh/HiggsAudioV3TTS` only indexes JSON/YAML metadata (0 symbols) because jcodemunch does not parse `.ipynb`. For symbol-level code search, use the temporary extracted-code index `local/higgs-tts-extracted-f5787a9f` (32 symbols from notebook cells).
4. **ast-grep** — `/usr/local/bin/ast-grep`. It cannot read `.ipynb` directly, so operate on the extracted Python file at `/tmp/higgs-tts-extracted/higgs_audio_v3_tts_colab_extracted.py`. Example:
   ```bash
   ast-grep run --pattern 'subprocess.run($$$ARGS)' --lang python /tmp/higgs-tts-extracted/higgs_audio_v3_tts_colab_extracted.py
   ```

If the notebook changes and you need fresh jcodemunch/ast-grep coverage, re-extract the code cells and re-index.

## Task completion checklist

Since there are no tests/linters, "done" means the notebook pipeline runs end-to-end:

- [ ] Cell 1 reports `sm_XX` with `XX >= 80`.
- [ ] Cell 2 completes; `sgl-omni` is on `PATH` and `sgl-omni serve --help` prints.
- [ ] Cell 3 downloads the model.
- [ ] Cell 4 starts the server on `127.0.0.1:8000` and `/health` or `/v1/models` responds within 15 minutes, with no protobuf/TensorFlow crash.
- [ ] Cell 5 launches Gradio and prints a public `gradio.live` link; TTS and voice-cloning tabs generate WAV files.
- [ ] Cell 6 cleanly shuts down Gradio and the server.

After any notebook edit, re-run the affected cells and at minimum the server + UI cells before considering the change complete.
