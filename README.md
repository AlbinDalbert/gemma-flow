# gemma-flow

Tiny WhisperFlow-ish dictation client for Hyprland + Wayland, backed by a self-hosted Ollama multimodal Gemma model.

Current target setup:

- Ollama: `http://ollama.lan`
- Model: `gemma4:12b`
- Recording: `pw-record` / PipeWire
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

## Hyprland bind

```ini
bind = $mainMod, T, exec, /home/chell/.local/bin/gemma-flow toggle
```

Press `Super+T` once to start listening, then again to stop. While processing, the focused window should stay focused; when the model returns, the text is pasted there.

## Notes

Ollama reports `gemma4:12b` capabilities as `completion`, `tools`, `thinking`, and `vision`, but `/api/show` exposes an audio projector:

- `clip.has_audio_encoder = true`
- `clip.has_vision_encoder = true`

The client sends WAV audio through Ollama's multimodal `images` field, which works against the tested Ollama server version `0.30.7`.
