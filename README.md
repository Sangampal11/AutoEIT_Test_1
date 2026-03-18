# AutoEIT — GSoC 2026 Test I Submission
### Audio-to-Text Transcription Pipeline for Spanish Elicited Imitation Task

---

## Overview

This project automates the transcription of Spanish EIT (Elicited Imitation Task) audio recordings. It segments participant responses, transcribes them using Whisper ASR, corrects ASR errors, and outputs results in the required Excel format.

**4 participants | 30 sentences each | 120 total utterances**

---

## Project Structure

```
AutoEIT-Test/
├── data/
│   ├── 038010_EIT-2A.mp3
│   ├── 038011_EIT-1A.mp3
│   ├── 038012_EIT-2A.mp3
│   ├── 038015_EIT-1A.mp3
│   └── AutoEIT Sample Audio for Transcribing.xlsx
├── src/
│   └── transcribe.py
├── outputs/
│   └── AutoEIT_Transcriptions_Completed.xlsx
├── requirements.txt
└── README.md
```

---

## Setup

```bash
# 1. Clone
git clone <your-repo-url>
cd AutoEIT-Test

# 2. Virtual environment
python -m venv .venv
.venv\Scripts\activate        # Windows
source .venv/bin/activate     # Mac/Linux

# 3. Install
pip install -r requirements.txt
```

**ffmpeg required (audio processing):**
- Windows: https://ffmpeg.org/download.html → add to PATH
- Mac: `brew install ffmpeg`
- Linux: `sudo apt install ffmpeg`

---

## Run

```bash
python src/transcribe.py
```

Output saved to `outputs/AutoEIT_ALL4_FINAL_Transcriptions.xlsx`

---

## Approach

### Step 1 — MP3 to WAV Conversion
Each MP3 converted to 16kHz mono WAV using ffmpeg.

### Step 2 — Silence-Based Segmentation
EIT recordings contain English instructions first, then 30 Spanish sentences.

- Skip instructions using EIT onset times:
  - 038010, 038011, 038015 → onset ~148s
  - 038012 → onset 720s (12:00 min, noted in template)
- ffmpeg silence detection: threshold −35dB, min silence 1.5s
- First 30 speech windows after onset = 30 participant responses
- Each saved as individual WAV segment

### Step 3 — Whisper ASR

```python
model.transcribe(wav_path,
    language="es",            # Force Spanish
    temperature=0,            # Deterministic output
    beam_size=5,              # Better accuracy
    no_speech_threshold=0.6   # Correctly marks silence
)
```

### Step 4 — ASR Error Correction

Two types of issues handled differently:

| Type | Action |
|------|--------|
| ASR error (Whisper mishears audio) | **Corrected** |
| Participant error (L2 grammar/vocab) | **Kept verbatim** |

Example ASR errors found and corrected:

| Participant | Item | Whisper (wrong) | Corrected |
|-------------|------|----------------|-----------|
| 038015 | 03 | `¿El carro no tiene pelo?` | `El carro lo tiene Pedro` |
| 038015 | 08 | `lleve mañana toro al día` | `llueva mañana todo el día` |
| 038015 | 18 | `de asustar su apartamento` | `de pintar su apartamento` |
| 038015 | 28 | `El prueba no fue difícil` | `El examen no fue tan difícil` |

### Step 5 — Excel Output

Results written into provided template. Color-coded for clarity:
- 🔵 Blue = ASR error corrected
- 🟣 Purple = Participant error noted (kept as-is)
- 🟠 Orange = Truncated response
- ⬜ Grey = No speech / silence

---

## Results

| Participant | File | Complete | ASR Fixed | Truncated | Silence |
|-------------|------|:--------:|:---------:|:---------:|:-------:|
| 038010 | EIT-2A | 23/30 | 0 | 5 | 2 |
| 038011 | EIT-1A | 26/30 | 0 | 2 | 2 |
| 038012 | EIT-2A | 16/30 | 0 | 0 | 14 |
| 038015 | EIT-1A | 12/30 | 18 | 0 | 0 |

038015 had 18 ASR errors — Whisper struggled most with this participant's L2 accent. All corrected by manual segment review.

038012 had 14 silence items — lowest proficiency participant, did not respond to many sentences.

---

## Transcription Conventions

| Symbol | Meaning |
|--------|---------|
| `...` | Pause or trailing off |
| `x-` | False start |
| `[silence]` | No speech detected |
| `[inaudible]` | Speech present but unclear |

**Core rule:** Participant grammar/vocabulary errors are NEVER corrected. Only Whisper ASR errors are fixed.

---

## Challenges

### 1. Mixed audio channel
Stimulus audio and participant response are on the same mono channel.
**Fix:** Responses always follow a silence gap (listen → speak). Silence detection after EIT onset captures only responses.

### 2. 038012 starts at 12:00
**Fix:** EIT onset manually set to 720s for this participant per template note.

### 3. Whisper errors on L2 Spanish
Whisper trained on native speech — makes systematic phonetic errors on accented L2 speech.
**Fix:** Manual review of every segment. ASR errors corrected, participant errors kept.

### 4. Disfluency suppression
Whisper removes filled pauses (um, uh, este) and false starts.
**Fix:** Truncated/flagged items noted in Excel for human review.

---

## Evaluation

### Automatic metrics (vs gold transcripts)
- **WER** (Word Error Rate)
- **CER** (Character Error Rate) — more sensitive to Spanish morphology

### Manual
- Re-listen to flagged segments
- Confirm participant errors are preserved
- Verify silence items

### Whisper WER benchmarks

| Model | Native Spanish | L2 Spanish |
|-------|:--------------:|:----------:|
| medium | ~6% | ~15–20% |
| large-v3 | ~4% | ~8–12% |

Current pipeline uses `medium`. Upgrading to `large-v3` recommended for production.

---

## Future Work

1. Whisper `large-v3` for better L2 accuracy
2. Disfluency restoration via word-level timestamps
3. Speaker diarization to separate stimulus vs response
4. Fine-tune on L2 Spanish EIT data
5. Auto-confidence scoring for flagging uncertain segments

---

## Dependencies

| Package | Purpose |
|---------|---------|
| `openai-whisper` | ASR |
| `ffmpeg` (system) | Audio conversion + silence detection |
| `scipy` | WAV I/O |
| `numpy` | Signal processing |
| `openpyxl` | Excel output |
| `pandas` | Data handling |
EOF
Output


