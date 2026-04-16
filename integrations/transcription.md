# Transcription Integration (Voice Notes + Audio Files)

## What it does

Transcribes audio files (voice notes, call recordings, any audio format) to text locally using faster-whisper. No cloud service, no API key needed. Runs on Apple Silicon and Intel Macs.

## Auth

None. The model downloads from HuggingFace on first use (no authentication required for default models). Runs entirely locally after the initial download.

## Setup

```bash
pip install faster-whisper
```

The `small` model (~460MB) downloads automatically on first transcription. For higher accuracy, use `medium` (~1.5GB) or `large-v3` (~3GB).

## Common Operations

### Transcribe a single file

```python
from faster_whisper import WhisperModel

model = WhisperModel('small', compute_type='int8')
segments, info = model.transcribe('path/to/audio.ogg', language='es')

print(f'Language: {info.language}')
for seg in segments:
    print(seg.text)
```

### Transcribe all files in a directory

```python
from faster_whisper import WhisperModel
import os

model = WhisperModel('small', compute_type='int8')

for f in sorted(os.listdir('audio_files')):
    if f.endswith(('.ogg', '.wav', '.mp3', '.m4a')):
        segments, info = model.transcribe(os.path.join('audio_files', f), language='es')
        text = ' '.join(seg.text.strip() for seg in segments)
        print(f'=== {f} ===')
        print(text)
        print()
```

### Save transcription to a text file alongside the audio

```python
from pathlib import Path
from faster_whisper import WhisperModel

model = WhisperModel('small', compute_type='int8')

audio_path = Path('recording.ogg')
segments, _ = model.transcribe(str(audio_path), language='es')
text = ' '.join(seg.text.strip() for seg in segments)

transcript_path = audio_path.with_suffix('.txt')
transcript_path.write_text(text)
print(f'Saved: {transcript_path}')
```

## Model options

| Model | Size | Speed | Accuracy | Best for |
|-------|------|-------|----------|----------|
| `tiny` | ~75MB | Fastest | Low | Quick previews, noisy audio |
| `small` | ~460MB | Fast | Good | Voice notes, clear speech |
| `medium` | ~1.5GB | Medium | Better | Meetings, multiple speakers |
| `large-v3` | ~3GB | Slow | Best | Important recordings, accented speech |

Use `compute_type='int8'` for faster inference with minimal quality loss on CPU / Apple Silicon.

## Supported formats

`.ogg`, `.wav`, `.mp3`, `.m4a`, `.flac`, `.webm` — faster-whisper uses ffmpeg internally, so anything ffmpeg can decode works.

## Language detection

If you don't specify a language, the model auto-detects. However, specifying the language (e.g., `language='es'` for Spanish, `language='en'` for English) significantly improves accuracy and speed.

## Gotchas

- First run downloads the model from HuggingFace. The `small` model is ~460MB. Subsequent runs use the cached model (stored in `~/.cache/huggingface/`).
- `compute_type='int8'` is recommended for CPU and Apple Silicon. Use `float16` only with CUDA GPUs.
- For very long audio (>30 min), faster-whisper streams the transcription so memory stays bounded.
- Whisper can hallucinate on silent or very noisy segments (repeating words, inserting phrases that weren't said). If you see suspicious repetition in transcripts, trim silent segments before transcribing.
- Voice notes from Telegram are `.ogg`, from WhatsApp are `.opus` wrapped in `.ogg`. Both work out of the box.
- The model is multilingual. It handles code-switching (mixing languages in one audio) reasonably well but not perfectly.
