# gemma-flow

Tiny WhisperFlow-ish dictation client for Hyprland + Wayland, backed by a self-hosted Ollama multimodal Gemma model.

Current target setup:

- Ollama: `http://ollama.lan`
- Model: `gemma4:12b`
- Recording: `pw-record` / PipeWire
- Upload: converts the WAV to FLAC before sending, to avoid proxy/body-size issues on longer clips
- Transcription modes: `gemma_direct`, `gemma_two_pass`, or `whisper_gemma`
- Status UI: `notify-send` / mako
- Insertion: `wl-copy` + `hyprctl dispatch sendshortcut ...`, then restore previous clipboard payload
  - auto uses `Ctrl+V`, or `Ctrl+Shift+V` for known terminal classes

## Commands

```bash
./bin/gemma-flow probe          # verify Ollama/model/audio attachment support
./bin/gemma-flow toggle         # start recording; run again to stop/process/paste
./bin/gemma-flow start          # start recording
./bin/gemma-flow stop           # stop/process/paste
./bin/gemma-flow stop --no-paste
./bin/gemma-flow cancel
./bin/gemma-flow status
./bin/gemma-flow transcribe-file ./audio.wav
./bin/gemma-flow write-config   # create ~/.config/gemma-flow/config.toml
```

## Transcription modes

Pick one in `~/.config/gemma-flow/config.toml`:

```toml
transcription_mode = "gemma_two_pass"  # choose one:
# "gemma_direct"    = audio -> Gemma final text
# "gemma_two_pass"  = audio -> Gemma raw transcript -> Gemma cleanup
# "whisper_gemma"   = local whisper.cpp raw transcript -> Gemma cleanup
```

`whisper_gemma` requires a local whisper.cpp binary/model. Configure with:

```toml
whisper_model = "base.en"
# or explicitly:
# whisper_model_path = "/home/chell/.local/share/whisper.cpp/ggml-base.en.bin"
```

For project names or words Whisper often mishears, add hints:

```toml
cleanup_vocabulary = ["Gemma", "Ollama", "Whisper", "whisper.cpp", "Hyprland"]

[phrase_replacements]
"Jammer" = "Gemma"
"Jemma" = "Gemma"
```

These hints are passed to Whisper as initial context and to Gemma's cleanup prompt.

## Hyprland bind

```ini
bind = $mainMod, T, exec, /home/chell/.local/bin/gemma-flow toggle
```

Press `Super+T` once to start listening, then again to stop. While processing, the focused window should stay focused; when the model returns, the text is pasted there.

## Notes

Ollama reports `gemma4:12b` capabilities as `completion`, `tools`, `thinking`, and `vision`, but `/api/show` exposes an audio projector:

- `clip.has_audio_encoder = true`
- `clip.has_vision_encoder = true`

The client sends audio through Ollama's multimodal `images` field, which works against the tested Ollama server version `0.30.7`.
