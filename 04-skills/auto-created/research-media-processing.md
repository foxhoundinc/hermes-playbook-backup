---
name: research-media-processing
category: research
description: Process EPUB/PDF/MP3 research materials and publish to Discord threads. Handles extraction without external deps, audio transcription, and threaded delivery.
---

# Research Media Processing

When user uploads EPUB/PDF/MP3 files for deep research analysis in Discord threads.

## Workflow Steps

### 1. RECEIVE FILES

**If user has headless Linux + Windows PC:**
- Guide them to use **WinSCP** (free, SFTP)
- Have them fill in: Host (Linux IP via `hostname -I`), Port 22, Username
- Create target folder: `mkdir -p ~/Documents/research_media`
- User drags files from Windows (left panel) → Linux (right panel)
- Files land in `~/Documents/research_media/`

**Alternative transfer methods:**
- Cloud sync (Google Drive/Dropbox → download from Linux)
- SCP/SSH from terminal (same network)

**Check files:**
```bash
ls -lh ~/Documents/research_media/
```

**Expected formats:** `.epub`, `.pdf`, `.mp3`

---

### 1b. TELEGRAM GROUP SETUP (Optional but recommended)
For organization beyond Discord, set up a Telegram Group with Topics:
1. User creates Telegram Group → toggles **Topics** ON
2. Add bot to group with **Admin** permissions (needed to create topics)
3. Send a message in the group so bot can capture Chat ID
4. Bot finds Chat ID via `getUpdates` API (or `my_chat_member` update)
5. Bot creates topics via `createForumTopic` API:
```python
# Create topic in forum group
headers = {"Content-Type": "application/json"}
for topic in ["Command Center", "Research", "Media Processing"]:
    resp = requests.post(f"{tg_api}/createForumTopic", 
                        json={"chat_id": chat_id, "name": topic})
```
6. Topics act like Discord channels - separate context, never delete

### 2. EXTRACT EPUB/PDF TEXT (RECOMMENDED: PyMuPDF + ebooklib)

**Install dependencies once in the Hermes venv:**
```bash
cd ~/.hermes/hermes-agent && source venv/bin/activate
pip install PyMuPDF ebooklib chardet
```
These work in the Hermes venv with ZERO system dependencies (no pdftotext, no Calibre).

**EPUB Extraction (tested on 5+ books, ~10 sec per book):**
```python
import ebooklib
from ebooklib import epub
import re

book = epub.read_epub(epub_path)
text_parts = []
for item in book.get_items_of_type(ebooklib.ITEM_DOCUMENT):
    text_parts.append(item.get_content().decode('utf-8', errors='ignore'))

full_text = "\n\n".join(text_parts)
clean = re.sub(r'<[^>]+>', ' ', full_text)
clean = re.sub(r'\s+', ' ', clean).strip()
# Write to output file with open(output_path, 'w') as f: f.write(clean)
```

**PDF Extraction (tested on 5+ books, ~1 min per book):**
```python
import fitz  # PyMuPDF

doc = fitz.open(pdf_path)
text_parts = []
for page in doc:
    text = page.get_text()
    if text.strip():
        text_parts.append(text)
doc.close()
full_text = "\n\n".join(text_parts)
# Write with open(output_path, 'w') as f: f.write(full_text)
```

**Batch extraction — process all books in `~/Documents/research_media/`:**
```python
import os
media_dir = "/home/cjhermes/Documents/research_media"
output_dir = os.path.join(media_dir, "extracted_books")
os.makedirs(output_dir, exist_ok=True)

for filename in sorted(os.listdir(media_dir)):
    if filename.endswith('.pdf'):
        words = extract_pdf(os.path.join(media_dir, filename), os.path.join(output_dir, filename.replace('.pdf', '.txt')))
    elif filename.endswith('.epub'):
        words = extract_epub(os.path.join(media_dir, filename), os.path.join(output_dir, filename.replace('.epub', '.txt')))
```
Extracted 9 books (915,000 total words) in under 10 seconds combined.

### 4. TRANSCRIBE AUDIO (MP3 → TEXT) — CHUNKED APPROACH

**CRITICAL: Always use chunked transcription for audiobooks >1 hour.** The monolithic approach (transcribing whole MP3 at once) **crashes silently** on CPU — logs stop after model loading with no output. Chunking into 10-minute segments is the only reliable method on CPU-only systems.

**Install dependencies:**
```bash
uv pip install --python ~/.hermes/hermes-agent/venv/bin/python3 faster-whisper pydub
# ffmpeg must also be installed on the system
```

**Chunked transcription script** — creates this file and runs it:

```python
#!/usr/bin/env python3
"""
Chunked Audio Transcription — splits MP3s into 10-min chunks,
transcribes each with faster-whisper, stitches results together.
Saves progress after each chunk so crashes don't lose work.
"""
import os, sys, json, time
from pathlib import Path
from pydub import AudioSegment
from faster_whisper import WhisperModel

MEDIA_DIR = "/home/cjhermes/Documents/research_media"
TRANSCRIPTS_DIR = os.path.join(MEDIA_DIR, "transcripts")
LOG_FILE = "/tmp/audio_transcription.log"
CHUNK_DURATION_MS = 10 * 60 * 1000  # 10 minutes

for d in [TRANSCRIPTS_DIR, os.path.join(TRANSCRIPTS_DIR, "chunks"),
          os.path.join(TRANSCRIPTS_DIR, "chunks_stitched"),
          os.path.join(TRANSCRIPTS_DIR, "chunks_raw")]:
    os.makedirs(d, exist_ok=True)

def log(msg):
    with open(LOG_FILE, "a") as f:
        f.write(msg + "\n")
    print(msg, flush=True)

def split_into_chunks(mp3_path, output_dir):
    audio = AudioSegment.from_mp3(mp3_path)
    total = len(audio)
    n_chunks = (total // CHUNK_DURATION_MS) + 1
    paths = []
    for i in range(n_chunks):
        name = f"chunk_{i:04d}.wav"
        path = os.path.join(output_dir, name)
        paths.append(path)
        if os.path.exists(path):
            continue
        chunk = audio[i*CHUNK_DURATION_MS:(i+1)*CHUNK_DURATION_MS]
        chunk.export(path, format="wav")
    return paths

def main():
    mp3s = sorted(f for f in os.listdir(MEDIA_DIR) if f.endswith(".mp3"))
    if not mp3s:
        log("No MP3 files found!"); sys.exit(1)
    
    log("Loading model...")
    model = WhisperModel("base", device="cpu", compute_type="int8")
    log(f"Found {len(mp3s)} MP3 files")
    
    for mp3_name in mp3s:
        mp3_path = os.path.join(MEDIA_DIR, mp3_name)
        safe_name = "".join(c if c.isalnum() or c in " _-" else "_" for c in mp3_name[:50])
        wav_dir = os.path.join(TRANSCRIPTS_DIR, "chunks_raw", safe_name)
        os.makedirs(wav_dir, exist_ok=True)
        
        log(f"\nProcessing: {mp3_name}")
        chunks = split_into_chunks(mp3_path, wav_dir)
        log(f"  Split into {len(chunks)} chunks")
        
        progress_path = os.path.join(TRANSCRIPTS_DIR, "chunks", f"{safe_name}_progress.json")
        stitched_file = os.path.join(TRANSCRIPTS_DIR, "chunks_stitched", f"{safe_name}.txt")
        
        transcribed = set()
        parts = []
        if os.path.exists(progress_path):
            with open(progress_path) as f:
                data = json.load(f)
            transcribed = set(data.get("completed_chunks", []))
            parts = data.get("partial_text", [])
        
        for i, chunk_path in enumerate(chunks):
            key = f"chunk_{i:04d}"
            if key in transcribed:
                continue
            log(f"  [{i+1}/{len(chunks)}] Transcribing...")
            segs, _ = model.transcribe(chunk_path, beam_size=1)
            text = " ".join(s.text.strip() for s in segs if s.text.strip())
            if text:
                parts.append(text)
                with open(os.path.join(TRANSCRIPTS_DIR, "chunks", f"{safe_name}_part_{i:04d}.txt"), "w") as f:
                    f.write(text)
            with open(progress_path, "w") as f:
                json.dump({"completed_chunks": [f"chunk_{j:04d}" for j in range(i+1)],
                           "partial_text": parts, "total_chunks": len(chunks)}, f)
            with open(stitched_file, "w") as f:
                f.write("\n\n".join(parts))
            log(f"  [{i+1}/{len(chunks)}] DONE ({len(text)} chars)")
        
        log(f"  COMPLETE: {stitched_file} ({len(' '.join(parts).split())} words)")

if __name__ == "__main__":
    main()
```

**Run in background:**
```bash
nohup python3 transcribe_chunked.py > /tmp/audio_transcription.log 2>&1 &
echo "PID: $!"
```

**Transcription speed guide:**
- 7hr audiobook: ~2 hours on CPU
- 16hr audiobook: ~5+ hours on CPU
- Always run in background with `nohup`
- Monitor: `tail -f /tmp/audio_transcription.log`

### 5. UPLOAD TO DISCORD THREAD

Get thread ID from user, then:

```python
import requests, yaml

with open("~/.hermes/config.yaml") as f:
    config = yaml.safe_load(f)
token = config['discord']['token']

url = f"https://discord.com/api/v10/channels/{thread_id}/messages"
headers = {"Authorization": f"Bot {token}"}

with open(text_file, 'rb') as f:
    resp = requests.post(url, headers=headers, 
                        data={"content": f"📚 Extracted text from {book_title}"},
                        files={"files[0]": (filename, f, "text/plain")})
```

### 6. ORGANIZE DISCORD STRUCTURE

**CRITICAL:** Discord threads cannot contain sub-threads. For complex projects:

**Option A - Categories (best for permanent organization):**
- Create a Category with multiple channels inside it
- **WARNING:** Bot needs `Manage Channels` permission to create categories/channels
- **If bot can't manage:** Guide user to create manually via Discord UI:
  1. Right-click Server Name → Create Category → Name it
  2. Right-click the Category title → Create Channel → Repeat 5-6x

**Option B - Master Thread (simpler):**
- Use one thread with pinned messages as table of contents
- Tag each upload with clear prefixes: `[NEUROSCIENCE]`, `[SKILL ACQUISITION]`

### 7b. DUAL-PLATFORM ORGANIZATION

For projects spanning both Discord and Telegram:
- **Discord**: Deep research, organized media storage, file hosting, long-form analysis
- **Telegram**: Quick commands, status updates, mobile-friendly, cross-session memory
- **Telegram Groups with Topics** act like Discord channels (separate context per topic = better memory recall)
- Set up Telegram Group first, add bot as admin, create matching topics
- Send a message in the group so bot can capture Chat ID for API access

### 8. FINDING GUILD ID FOR DISCORD API
The guild_id in memory/config can be stale/wrong. To find the actual one reliably:
```python
# Use a known channel ID to find the actual guild
resp = requests.get(f"https://discord.com/api/v10/channels/{known_channel_id}", headers=headers)
guild_id = resp.json().get('guild_id')  # This is ALWAYS correct
```

### 8. ARCHIVE.ORG BOOK BORROWING & DOWNLOAD
When sourcing books from archive.org for the research pipeline:

**Login + Borrow Flow:**
- Navigate to `https://archive.org/account/login`
- Fill in email/password → click login
- **Book may already be borrowed** (shows "Return now" instead of "Borrow")
- "Borrow" = click to start loan (auto-renews with use)

**Download Check:**
- Go to `https://archive.org/download/{identifier}/`
- **Lock icon** = Restricted (DRM protected, can't download)
- **No lock** = Free to download

**DRM Handling:**
- Modern copyrighted books on archive.org are usually DRM-restricted
- EPUB/PDF "Restricted" → cannot download directly
- LCP format available but also DRM-protected
- **Workaround:** Read online via BookReader (`/stream/{identifier}`) and extract text manually/chapter by chapter
- **Alternative:** Source author's original academic papers (free, same science, extractable)

### 9. TELEGRAM GROUP CHAT ID DISCOVERY
Group IDs come in formats that need prefix adjustments. Try all three:
```python
raw = "1234567890"
for cid in [raw, f"-{raw}", f"-100{raw}"]:
    resp = requests.post(f"{api}/createForumTopic", json={"chat_id": cid, "name": "Test"})
    if resp.json().get("ok"):
        print(f"Correct ID format: {cid}")
        break
```
**Correct format for supergroups:** `-100` prefix + raw ID

### 7. VERIFY UPLOAD

Message user with:
- File sizes
- Word counts
- Which thread/channel received the upload
- ETA for any remaining audio transcriptions

## Pitfalls

- **EPUB filenames often have special chars** - use `ls` to get exact name, don't hardcode
- **No pip in some environments** - use `uv pip install --python ~/.hermes/hermes-agent/venv/bin/python3` not system pip
- **Zipfile extraction** works universally, unlike ebooklib which requires deps
- **Discord API guild_id** can be wrong - find it via `/channels/{known_channel_id}` first, then read `guild_id` from response
- **Discord bot permissions:** Creating categories/channels requires `Manage Channels` permission - if grayed out, guide manual setup
- **Threads cannot nest** - don't create threads inside threads - use categories/channels or master thread approach
- **CPU transcription** is slow but reliable - set expectations (7hr audio ≈ 2hr CPU)
- **Files over 25MB** can't be uploaded directly to Discord - must extract text first
- **faster-whisper:** Use `device="cpu", compute_type="int8"` when no GPU available - CUDA will fail gracefully
- **faster-whisper MONOLITHIC TRANSCRIPTION CRASHES:** Transcribing entire audiobook files at once on CPU silently crashes/fails — logs stop after model loading with zero output. ALWAYS use chunked approach (10-min segments) for audiobooks >1 hour. Install `pydub` dependency: `uv pip install --python ~/.hermes/hermes-agent/venv/bin/python3 pydub`
- **Archive.org:** Modern copyrighted books are DRM-locked (lock icon on download page). EPUB/PDF "Restricted" = cannot download. Workarounds: (1) extract text from online BookReader chapter by chapter, (2) source author's original academic papers (free, same content), (3) user sources locally → SCP transfer. Always check `/download/{identifier}/` first to see lock status before attempting borrow
- **Discord guild_id stale:** guild_id in memory/config can be wrong after server recreation. Always verify by calling `GET /channels/{known_channel_id}` first, then read `guild_id` from the response
- **Telegram group Chat IDs:** Raw IDs like `1234567890` need the `-100` prefix for supergroups. Try all three formats: `raw`, `-raw`, `-100raw` — the `-100raw` format is correct for topic-enabled groups
- **Dual-platform memory:** Telegram Groups with Topics create separate context per topic, which improves cross-session memory recall. Use Discord for deep file work, Telegram for quick status/commands
- **browser_vision for state inspection:** Before clicking interactive elements on complex pages (Archive.org, login forms), use `browser_vision` to confirm button states, loading indicators, and whether actions already succeeded