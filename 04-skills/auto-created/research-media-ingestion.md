---
name: research-media-ingestion
category: research
description: Download, process, and publish research media (books, audiobooks, papers) to Discord for ENHANCED MIND platform
---

# Research Media Ingestion

Full workflow for acquiring research materials (books, audiobooks, academic papers, articles, videos) and delivering processed content to Discord.

## What We Know Works (Tested Apr 2026)

## Prerequisites

- **Linux box**: Headless machine running Ubuntu/Debian with Python 3.11+
- **Storage**: `~/Documents/research_media/` for raw files
- **Venv**: `/home/cjhermes/.hermes/hermes-agent/venv/` with `faster-whisper`, `pydub`, `ffmpeg`
- **Discord**: Bot connected, category with channels set up
- **Hermes Agent**: Running with Discord integration, Qwen 3.6 Plus model (1M context window)

## Book Sourcing Strategy

### Priority Sources (check in order):

1. **User files** — User transfers via WinSCP to `~/Documents/research_media/`
2. **Direct OA Journals (WORKS)** — PLOS ONE, Frontiers in Psychology, Nature Reviews OA, OpenStax
3. **Internet Archive** — Search `archive.org`, but many are borrow-only (DRM). DO NOT waste time trying to download — they are all 401/403 locked.
4. **Europe PMC** — Useful for searching, but PDF downloads are BLOCKED (403/stream reset). Only 2 of 13 tested papers had full-text XML available.
5. **PMC/NCBI OA API** — Returns "idIsNotOpenAccess" even for papers with PMCID. Does NOT work for bulk download.
6. **arXiv** — NOT relevant for cognitive/psychology/sports topics (zero results). Skip.
7. **Unpaywall API** — Returns metadata but PDF URLs are often still paywalled or return HTML.

### Working Download Pattern for Papers:

```bash
# PLOS ONE — always works with this pattern
curl -L -s -o output.pdf "https://journals.plos.org/plosone/article/file?id=10.1371/journal.pone.XXXXXXX&type=printable"

# Frontiers in Psychology — works
curl -L -s -o output.pdf "https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.YEAR.XXXXX/pdf"

# Nature Reviews — works
curl -L -s -o output.pdf "https://www.nature.com/articles/nrnXXXXX.pdf"

# OpenStax textbooks — always free
curl -L -s -o output.pdf "https://assets.openstax.org/oscms-prodcms/media/documents/BookName-WEB.pdf"
```

- User feeds URLs → agent extracts full text → saves to research_media articles/ → uploads to Discord channel
- Web search via Hermes is broken (CAPTCHA on DuckDuckGo/Google, empty results from EPMC OA queries). Until web search is fixed with proper API keys, rely on user-provided URLs and direct publisher endpoints.
- **Flow State Query Fix (IMPORTANT):** When searching PubMed for "flow state", bare query `"flow state"[tiab]` returns medical papers about blood flow physiology. ALWAYS use qualifier: `"flow state"[tiab] AND psychology[tiab]` or the backup: `dispositional flow[tiab] OR peak experience[tiab]`

## Paper Sourcing Strategy (tested and revised)

### What DOESN'T work (all tested and failed):
- Downloading PDFs from Europe PMC or NCBI directly via API — returns 401/403/HTML instead of PDF, with stream resets
- Semantic Scholar API returns good metadata but no OA PDF links for most papers
- Europe PMC REST API full-text XML only available for a small subset (2/13 papers in our test)
- Arxiv doesn't have psychology/sports science papers
- DuckDuckGo and Google web searches block the agent with CAPTCHA

### What DOES work:

**Direct publisher OA PDF URLs (verified working):**
```
# PLOS ONE (always OA, direct PDF):
https://journals.plos.org/plosone/article/file?id=10.1371/journal.pone.0261492&type=printable

# Frontiers in Psychology (always OA):
https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2021.652340/pdf
https://www.frontiersin.org/journals/psychology/articles/10.3389/fpsyg.2018.01240/pdf

# MDPI Sports (always OA):
https://www.mdpi.com/2411-5142/8/3/77/pdf

# Nature Reviews (some OA):
https://www.nature.com/articles/nrn3238.pdf
```

**Europe PMC full-text XML (subset):**
- Endpoint: `https://www.ebi.ac.uk/europepmc/webservices/rest/{PMCID}/fullTextXML`
- Only works for ~15% of papers with PMCID
- Returns XML that must be converted to readable text

**Unpaywall API (free, WORKS — updated Apr 2026):**
- `https://api.unpaywall.org/v2/{DOI}?email=skynetnp3@gmail.com`
- Check BOTH `is_best_oa_location: true` AND `best_oa_location.url_for_pdf` exists
- Only ~30-40% of papers have OA versions, but this reliably identifies them
- Use a real email you check — Unpaywall uses it as an identifier for rate limiting
- Endpoint: `https://api.unpaywall.org/v2/{DOI}?email=YOUR_EMAIL` — GET request, no API key needed for basic use

**User-provided URLs → extraction:**
- Agent visits URL with browser, extracts full text, saves to research_media/articles/, uploads to Discord
- This is the highest-yield approach currently
- Example: Steph Curry FitLight vision training article

**Abstract enrichment (Apr 2026):**
- For any PubMed paper, fetch abstract via: `https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=pubmed&id={PMID}&rettype=abstract&retmode=text`
- Store first 500 chars — go into Discord digest AND JSON queue
- Use EFetch sparingly (1 request per paper, 0.5s delay between calls to NCBI)

### Download script locations:
- `download_oa_resources.py` — Downloads from known OA publisher endpoints (WORKS) — NOTE: hardcoded static list, NOT dynamic
- `download_papers_xml.py` — Extracts full text from Europe PMC XML (partial success)
- `upload_to_discord.py` — Uploads files with descriptions to Discord channels (WORKS 100%) — NOTE: hardcoded file list, needs manual updating
- `transcribe_chunked.py` — Audiobook transcription (WORKS)

### Better Download Queue Pattern (Apr 2026 — use this instead of hardcoded scripts):
Build a JSON queue from the discovery step: `~/.hermes/cron_output/research_queue.json`
```json
{
  "run_date": "YYYY-MM-DD",
  "papers": [
    {
      "pmid": "12345678",
      "doi": "10.xxxx/xxxxx",
      "title": "Paper Title",
      "journal": "Journal",
      "pub_date": "2026-04",
      "abstract": "first 500 chars...",
      "topic": "Deliberate Practice",
      "oa_pdf_url": "https://..." or null,
      "url": "https://pubmed.ncbi.nlm.nih.gov/12345678/"
    }
  ],
  "oa_count": N,
  "total_count": N
}
```
- Cron discovery step: queries Unpaywall per paper, writes queue JSON, posts digest to #research-log
- Separate download step: reads queue JSON, downloads only papers with `oa_pdf_url != null`
- Age filter: skip papers >18 months old (calculate from today)
- Deduplication: track seen PubMed IDs in `~/.hermes/papers_seen.txt`, arXiv IDs in `~/.hermes/arxiv_seen.txt`

## Discord Upload Pattern

Using bot token via curl multipart upload:
```bash
curl -s \
  -F "payload_json={\"content\": \"DESCRIPTION_WITH_MARKDOWN\"}" \
  -F "files[0]=@/path/to/file" \
  -H "Authorization: Bot $DISCORD_TOKEN" \
  "https://discord.com/api/v10/channels/$CHANNEL_ID/messages"
```

Rate limit: 5 uploads per 5 seconds per channel. Script uses 3-second delay between uploads.

## Channel Mapping (Verified Working):

| Discord Channel | Channel ID | Content Types |
|---|---|---|
| #core-theory | 1488772795281444894 | Foundational frameworks, deliberate practice theory, sustainable performance |
| #elite-performance | 1488772941679165552 | Applied performance science, attention training, HRV, sports psych |
| #neuroscience | 1488773022478241812 | Brain plasticity, sleep/memory, neuroplasticity, foundational biology |

## Audio Transcription Pipeline

### Setup:

```bash
cd /home/cjhermes/.hermes/hermes-agent
source venv/bin/activate
pip install faster-whisper pydub
```

### Run Transcription:

The script at `~/Documents/research_media/transcribe_chunked.py`:
1. Splits MP3s into 10-minute chunks using pydub+ffmpeg
2. Transcribes each chunk with faster-whisper (base model, CPU, int8)
3. Saves progress after each chunk (crash-resume)
4. Stitches all transcripts into single .txt file

```bash
cd ~/Documents/research_media
python3 transcribe_chunked.py > /tmp/transcription_$(date +%Y%m%d_%H%M%S).log 2>&1 &
```

### Output Locations:

- Raw WAV chunks: `~/Documents/research_media/transcripts/chunks_raw/`
- Text chunks: `~/Documents/research_media/transcripts/chunks/`
- Stitched transcripts: `~/Documents/research_media/transcripts/chunks_stitched/`

## Processing Workflow

### 1. EPUB Extraction

Already extracted: The Core by Aki Hintsa (98k+ words)

```bash
cd ~/Documents/research_media
python3 extract_epub.py "path/to/book.epub"
```

### 2. Audiobook Transcription

Monitor progress:
```bash
ls ~/Documents/research_media/transcripts/chunks/ | wc -l
tail -f /tmp/audio_transcription.log
```

### 3. Academic Papers (PDF)

Already collected in: `~/Documents/research_media/open_access/`

### 4. Upload to Discord

Use the Hermes Discord integration to upload processed text to relevant channels:

| Content Type | Discord Channel | Thread Strategy |
|---|---|---|
| Book full text | #core-theory | One thread per book |
| Paper summaries | #neuroscience, #elite-performance | Group by topic |
| Transcribed audiobooks | #research-log | One thread per book |
| Status updates | #core-theory | Pinned status board |

## Discord Channel IDs

Category: ENHANCED MIND
- #core-theory: 1488772795281444894
- #elite-performance: 1488772941679165552  
- #neuroscience: 1488773022478241812
- #skill-acquisition
- #platform-architecture
- #research-log

## Article/URL Extraction Workflow (Primary method now)

## Transcript Processing Workflow (NEW - Apr 2026, Phase 2 Research Sweep)

### Tim Ferriss Transcripts (GOLD MINE DISCOVERED)
- **Source**: tim.blog/2018/09/20/all-transcripts-from-the-tim-ferriss-show/
- **Pattern**: Direct PDF links at `tim.blog/wp-content/uploads/YYYY/MM/{episode}-guest.pdf`
- **Count**: 825+ episode transcripts available (143 on the index page, more on newer pages)
- **Method**: 
  1. `curl -sL "{url}" -o /tmp/tf.pdf`
  2. `pdftotext -layout /tmp/tf.pdf /tmp/transcripts/tim-ferriss-{name}.txt`
  3. Categorize and post to Discord
- **Priority episodes found**:
  - Josh Waitzkin #002 (87K chars), #148 (162K chars) — Art of Learning
  - Maria Popova #039 (127K), #092 (32K) — Brain Pickings/creativity
  - Ryan Holiday #004 (104K) — Stoicism
  - James Altucher #018 (103K) — Idea generation
  - Cal Fussman #145 (203K) — Interview craft
  - Tony Robbins #035, #037, #038 — Peak performance
  - Sleep, Mindfulness, Habits, Fear setting episodes

### Huberman Lab Transcripts
- **Source**: YouTube auto-generated captions (YouTube ID → youtube-transcript-api)
- **Status**: 11 transcripts extracted (~1.34M chars) via `youtube-transcript-api` before IP block
- **Cloud IP issue**: YouTube blocks HTTP 429 for caption API from cloud IPs (AWS/GCP)
- **Workaround**: Extract when available, otherwise user must provide transcript from their PC
- **Key episodes**: Focus protocols, neuroplasticity, Waitzkin (3h17m), Cal Newport, Robert Greene, mental training, psilocybin, ketamine, willpower

### Big Think Transcripts
- **Source**: bigthink.com/articles/ — full transcripts server-rendered in Alpine.js tabs or HTML content
- **Method**: 
  1. `curl -sL "{url}" -o /tmp/bt.html`
  2. Extract paragraphs from transcript-containing sections
- **Extracted**: Kotler flow state (3 transcripts), Suzuki anxiety, Waldinger Zen resilience, Davis creativity
- **Total**: ~60K chars of flow state content

### YouTube Transcript Challenge (CRITICAL KNOWLEDGE)
- **Problem**: YouTube blocks ALL caption/subtitle download APIs from cloud IPs with HTTP 429
- **Affected**: youtube-transcript-api, yt-dlp, DownSub, youtubetranscript.com, timedtext API
- **User Workaround**: Open YouTube in Chrome -> "Show transcript" -> copy -> save .txt
- **File drop**: Drop into `~/Documents/transcripts/` — watcher auto-processes and posts to Discord

### Transcript Auto-Watcher
- **Script**: `~/watch_transcripts.py` (runs in background)
- **Directory**: `/Documents/transcripts/`
- **Behavior**: Scans every 15 seconds for new .txt files
- **Process**: Auto-categorizes by content keywords, posts to correct Discord channel
- **Tracking**: Saved processed files in `~/.hermes/transcripts_processed.json`

### Discord Posting Pattern for Transcripts
```python
# Use Discord API to post in chunks (max 2000 chars per message)
# Split at paragraph boundaries, add continuation labels (1/3, 2/3, 3/3)
# Label with source type for easy reference
# Post to category-specific channels based on content
# Log complete processing status to #research-log
```

### Transcript Content Mapping (which channel gets what)
- **MILITARY/SPECIAL FORCES**: #elite-performance
- **SPORTS/ATHLETES**: #elite-performance
- **LEARNING/PROTOCOLS**: #skill-acquisition
- **PODCASTS/INTERVIEWS**: #skill-acquisition (or #core-theory for theory-heavy episodes)
- **FLOW/COGNITIVE THEORY**: #core-theory
- **NEUROSCIENCE/BRAIN**: #neuroscience
- **SYSTEM/ARCHITECTURE**: #platform-architecture
- **UNKNOWN/GENERAL**: #skill-acquisition (default)

When user provides URLs like blog articles, news pieces, or YouTube links:

1. **Extract text** using browser tool (browser_navigate + browser_snapshot)
2. **Save** to `~/Documents/research_media/articles/filename.txt`
3. **Upload** to appropriate Discord channel with curl multipart upload
4. **Map topic** to channel based on ENHANCED MIND pillars:
   - Sports vision/perception → #elite-performance or #neuroscience
   - Training protocols/drills → #skill-acquisition or #elite-performance
   - Foundational science → #core-theory
   - Technology/systems (FitLight, Senaptec) → Note for platform architecture

## Web Research Tools (CONFIGURED Apr 2026)

Both search backends are now configured in ~/.hermes/.env:
- **Firecrawl** (optional, non-critical): Free plan historically offered 500 credits. Currently NOT the active web backend.
  - Our preferred web backend is `browser` mode (zero cost) + Parallel API
  - Only needed if you want to switch to Firecrawl for search/extract (via `hermes tools`)
- **Parallel**: Pay-per-query (~$0.08/search). Higher accuracy than competitors across benchmarks.
  - Signup: https://platform.parallel.ai/
  - Key: `PARALLEL_API_KEY=in9s...` (no monthly cost)

The `web` toolset is enabled for Discord and Telegram platforms. Web search and extraction now work without CAPTCHA blocks.

### Fallback if web tools unavailable:
- DuckDuckGo/Google hit CAPTCHA on agent browsing
- Europe PMC API returns 401/403 for PDF downloads but XML works for ~15% of papers
- User-provided URLs → agent extracts via browser tool

## Current Research Media Library (~Apr 2026)

### Books (partial):
- The Core by Aki Hintsa — Full text extracted (98k words)
- Regenerative Performance audiobook — Transcription in progress
- The Stimulated Mind audiobook — Queued
- **Priority books NOT acquired (DRM-protected):** Peak, Talent Code, Brain That Changes Itself, Relentless, Moonwalking with Einstein, Limitless, Endure, Can't Hurt Me

### Papers (downloaded + uploaded):
- PLOS ONE: Deliberate Practice (1.5MB)
- PLOS ONE: Neuroplasticity & Motor Learning (1MB)
- Frontiers: Attention Training (919KB)
- Nature Reviews: Sleep & Memory (196KB)
- Europe PMC XML: HRV Metrics, Training-Injury Prevention
- NIH: Brain Basics
- OpenStax: Intro Psychology

### Articles:
- Stephen Curry FitLight vision training (extracted from Eyedolatry Blog)
## ENHANCED MIND Priority Hunt List

Books to source (in priority order):
1. Peak — Anders Ericsson (deliberate practice)
2. The Talent Code — Daniel Coyle (myelin science)
3. The Brain That Changes Itself — Norman Doidge (neuroplasticity)
4. Relentless — Tim Grover (elite performance coaching)
5. Moonwalking with Einstein — Joshua Foer (memory)
6. Flow — Mihaly Csikszentmihalyi (flow state)
7. Limitless — Jim Kwik (brain training)
8. Endure — Alex Hutchinson (performance limits)
9. Can't Hurt Me — David Goggins (mental toughness)
10. The Power of Full Engagement — Jim Loehr (energy management)
