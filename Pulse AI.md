# Pulse AI — Technical Specification
### *An Autonomous Viral-News AI Personality for YouTube & X*
**Hackathon:** Global AI Hackathon Series · Track 2 — AI Showrunner
**Stack:** Qwen3.7-Max · HappyHorse 1.0 · Wan 2.6 · fal.ai · Alibaba Cloud DashScope

---

## 1. Vision & Concept

**Pulse AI** is a fully autonomous AI agent that, once triggered weekly, executes the entire production pipeline for a short-form viral news broadcast — from crawling the internet for trending content, to delivering a finished, broadcast-quality video narrated by a synthetic AI news personality named **ARIA** *(Autonomous Realtime Intelligence Anchor)*.

ARIA is not a flat text-to-speech robot. She is an expressive, emotionally resonant on-camera personality — warm, witty, occasionally sardonic — who *performs* the news rather than merely reading it. She reacts visibly to shocking statistics, leans into absurdity with a raised eyebrow, and closes every episode with a signature sign-off. The aesthetic is part late-night news desk, part digital influencer.

> **Core differentiator:** Every component — research, editorial judgment, scriptwriting, storyboard direction, video generation, captioning, and upload — is orchestrated autonomously by a single agentic loop. Zero human intervention is required between weekly trigger and published video.

---

## 2. Hackathon Track Alignment

| Judging Criterion | How Pulse AI Addresses It |
|---|---|
| **Narrative Ability** | Qwen3.7 writes 3-act story arcs per segment; each story has a hook, build, and payoff |
| **Multimodal Orchestration** | Agent chains: web crawl → LLM analysis → script → storyboard → HappyHorse T2V/I2V → audio merge → FFmpeg edit |
| **Token Budget Efficiency** | Tiered summarisation keeps input tokens minimal; only final script generation uses full-context reasoning |
| **Output Quality** | HappyHorse 1080p with native audio + Wan 2.6 for B-roll cutaways; colour-graded in post |
| **Creativity** | Unique AI personality with consistent visual identity, performance direction cues, and comedic beats |

---

## 3. System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    PULSE AI ORCHESTRATOR                     │
│              (Qwen3.7-Max Agentic Harness)                  │
└──────────────────────────┬──────────────────────────────────┘
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
   [STAGE 1]         [STAGE 2]        [STAGE 3]
  Web Crawler        Analyser         Editorial
  & Aggregator       Agent            Director
  (Tool calls)    (Qwen3.7-Max     (Qwen3.7-Max
                   Thinking mode)   Instruct mode)
          │                │                │
          └────────────────┴────────────────┘
                           │
                    [STAGE 4 & 5]
                  Script + Storyboard
                  (Qwen3.7-Max w/
                   structured output)
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
        [STAGE 6A]                [STAGE 6B]
    HappyHorse 1.0              Wan 2.6 (B-roll)
   Anchor segments             Cutaway visuals
   (fal.ai API)              (DashScope API)
              │                         │
              └────────────┬────────────┘
                           ▼
                     [STAGE 7]
                   Post-Production
                   (FFmpeg + Qwen3.7
                    caption writer)
                           │
                           ▼
                     [STAGE 8]
                  Platform Publisher
                (YouTube Data API v3
                 + X API v2)
```

---

## 4. Agent Pipeline — Detailed Stage Specification

### Stage 1: Viral Content Aggregation (The Crawl)

**Objective:** Gather the week's top 50 viral content items across categories.

**Data Sources & Tool Calls:**

| Source | Method | Data Extracted |
|---|---|---|
| Reddit `/r/all/top` | REST API (OAuth2) | Title, score, comments, subreddit |
| Twitter/X Trending | X API v2 Trending endpoint | Hashtag + top tweets |
| Google Trends | SerpAPI / pytrends | Weekly search spike terms |
| YouTube Trending | YouTube Data API | Video title, view count delta |
| TikTok Creative Center | Scraper (Playwright headless) | Hashtag trends |
| NewsAPI.org | REST API | Top headline clusters |

**Output:** Raw JSON manifest of 50 candidate stories, each with:
```json
{
  "id": "story_001",
  "headline": "...",
  "source": "reddit/r/technology",
  "engagement_score": 142000,
  "sentiment_polarity": 0.72,
  "category": "tech|culture|politics|science|weird",
  "url": "...",
  "media_urls": ["..."],
  "raw_text_snippet": "..."
}
```

**Token Budget Stage 1:** ~2,000 tokens (tool call overhead only; data stored externally)

---

### Stage 2: Qwen3.7 Analysis Agent — Editorial Scoring

**Objective:** Score and rank the 50 stories for broadcast viability using multi-dimensional analysis.

**Model:** `qwen3.7-max` in **Thinking Mode** (complex reasoning required)

**System Prompt (abbreviated):**
```
You are ARIA's senior editorial producer. Your job is to assess 50 viral stories
and score each on:
1. VIRAL_VELOCITY (0-10): How fast is this spreading RIGHT NOW?
2. NARRATIVE_RICHNESS (0-10): Does this story have a beginning, middle, payoff?
3. VISUAL_POTENTIAL (0-10): Can we generate compelling visuals for this?
4. ARIA_FIT (0-10): Would this amuse, shock, or delight our audience?
5. UNIQUENESS (0-10): Is this genuinely novel vs. derivative coverage?

Return structured JSON. Think step-by-step before scoring each item.
```

**Scoring → Selection:** Top 5 stories selected by weighted composite score:
`final_score = 0.25*velocity + 0.20*narrative + 0.20*visual + 0.20*aria_fit + 0.15*uniqueness`

**Token Budget Stage 2:** ~8,000 tokens (50 summaries × ~150 tokens each + reasoning)

---

### Stage 3: ARIA Character Direction

**Objective:** Determine ARIA's emotional register and performance style for each story.

**Model:** `qwen3.7-max` in **Instruct Mode**

For each of the 5 selected stories, the agent assigns:

```json
{
  "story_id": "story_003",
  "aria_emotion": "bemused_sarcasm",
  "delivery_pace": "moderate_with_pause_for_effect",
  "facial_cues": ["raised_eyebrow", "slight_smirk", "head_tilt_right"],
  "energy_level": "7/10",
  "transition_style": "whip_cut",
  "b_roll_mood": "satirical_infographic"
}
```

**ARIA Personality Matrix:**

| Story Category | Default Emotion | Performance Notes |
|---|---|---|
| Tech absurdity | Bemused sarcasm | Side-eye to camera, dry wit |
| Heartwarming viral | Warm delight | Genuine smile, soft voice |
| Political drama | Theatrical gravitas | Slow lean forward, pause |
| Science breakthrough | Wide-eyed wonder | Rapid delivery, hand gestures |
| Celebrity chaos | Barely-contained amusement | Suppressed laugh, whisper |

**Token Budget Stage 3:** ~3,000 tokens

---

### Stage 4: Script Generation

**Objective:** Write the complete broadcast script — the heart of the pipeline.

**Model:** `qwen3.7-max` in **Thinking Mode** (structured output via JSON schema)

**Show Format:**

```
[INTRO — 15 sec]    ARIA cold open: signature greeting + episode teaser
[STORY 1 — 60 sec]  Hook → Build → Payoff → ARIA reaction
[STORY 2 — 60 sec]  same structure
[STORY 3 — 60 sec]  same structure
[STORY 4 — 45 sec]  slightly lighter fare
[STORY 5 — 30 sec]  The "weird corner" — deliberately absurd final story
[OUTRO — 20 sec]    ARIA sign-off + call-to-action
─────────────────
Total: ~5.5 minutes
```

**Master Script Prompt System:**

```
You are writing for ARIA, Pulse AI's anchor. ARIA speaks directly to camera in
a second-person intimate style — as if confiding in a smart friend over coffee.

WRITING RULES:
- Every sentence must be speakable. No dependent clauses over 20 words.
- Plant a curiosity gap in sentence 1 of every story.
- Vary sentence rhythm: short punch. Then a longer, more nuanced elaboration.
- ARIA references herself in third-person ONCE per episode (her quirk).
- Each story ends with ARIA's "Hot Take" — one sentence, max 12 words.
- Include [PAUSE] markers for dramatic effect (HappyHorse uses these as beat cues).
- Include [REACTION: {emotion}] markers for facial expression direction.
- End EXACTLY with the catchphrase: "Stay curious. Stay weird. I'm ARIA — and this has been your Pulse."

OUTPUT FORMAT: Structured JSON with segments array.
```

**Script JSON Schema:**

```json
{
  "episode_number": 1,
  "episode_title": "...",
  "total_duration_seconds": 330,
  "segments": [
    {
      "segment_id": "intro",
      "duration_seconds": 15,
      "script_text": "...",
      "performance_cues": ["..."],
      "visual_direction": "...",
      "b_roll_needed": false
    },
    {
      "segment_id": "story_1",
      "story_headline": "...",
      "duration_seconds": 60,
      "script_text": "...",
      "performance_cues": ["[REACTION: wide_eyes]", "[PAUSE: 1s]"],
      "visual_direction": "Close-up on ARIA face, cut to B-roll at :20",
      "b_roll_prompt": "Cinematic shot of [SUBJECT] in [STYLE]...",
      "b_roll_duration_seconds": 8,
      "hot_take": "..."
    }
  ],
  "sign_off": "Stay curious. Stay weird. I'm ARIA — and this has been your Pulse."
}
```

**Token Budget Stage 4:** ~12,000 tokens (most expensive stage — justified by highest output value)

---

### Stage 5: Storyboard Direction

**Objective:** Convert the script into shot-by-shot visual production specs for HappyHorse.

**Model:** `qwen3.7-max` in **Instruct Mode**

For each script segment, the agent generates a HappyHorse-optimised text prompt using cinematographic language:

**HappyHorse Prompt Formula:**
```
[SUBJECT]: [ARIA / B-roll subject]
[SHOT TYPE]: Close-up / Medium / Wide / OTS (over-the-shoulder)
[LIGHTING]: [Mood description — e.g., "warm studio key light, cool fill, subtle rim light"]
[BACKGROUND]: [News desk / abstract gradient / subject-contextual environment]
[MOTION]: [Camera movement + ARIA body language]
[EMOTION/PERFORMANCE]: [Direct HappyHorse facial expression directive]
[AUDIO DIRECTION]: [Sync dialogue / ambient sound / music bed]
[DURATION]: [seconds]
[ASPECT RATIO]: 16:9
[RESOLUTION]: 1080p
```

**Example — ARIA Intro Shot:**
```
Subject: ARIA, a poised AI news anchor with luminous skin, silver-blue eyes,
sharp geometric earrings, wearing a structured navy blazer.
Shot type: Medium close-up, slight dutch angle resolving to level.
Lighting: Broadcast studio — warm golden key light from camera-left,
cool blue fill from right, subtle lens flare on camera move.
Background: Sleek dark glass news desk, soft bokeh city skyline through
floor-to-ceiling windows, animated pulse-wave logo floating lower-third.
Motion: ARIA leans slightly forward from relaxed position, eyes meet camera
with a knowing half-smile.
Performance: Confident, warm, slightly conspiratorial.
Audio: Sync dialogue — ARIA speaks opening line. Music bed: subtle electronic
pulse, 80 BPM, fades under voice at 2s.
Duration: 15 seconds. Aspect: 16:9. Resolution: 1080p.
```

**Token Budget Stage 5:** ~5,000 tokens

---

### Stage 6A: HappyHorse 1.0 — Anchor Video Segments

**API Provider:** fal.ai (official HappyHorse partner)

**Endpoint:** `POST https://fal.run/fal-ai/happyhorse/text-to-video`

**Request Pattern per segment:**

```python
import fal_client

def generate_anchor_clip(storyboard_prompt: str, duration: int) -> str:
    """Returns URL of generated video clip."""
    result = fal_client.subscribe(
        "fal-ai/happyhorse",
        arguments={
            "prompt": storyboard_prompt,
            "duration": duration,          # 3-15 seconds per call
            "resolution": "1080p",
            "aspect_ratio": "16:9",
            "audio_generation": True,      # Native joint audio-video
            "audio_type": "dialogue_sync", # Lip-sync mode
            "dialogue_text": segment["script_text"],
            "language": "en",
        }
    )
    return result["video"]["url"]
```

> **Critical Design Note:** HappyHorse generates up to **15 seconds per clip**. Long anchor segments (e.g., 60-second story reads) are decomposed into **4 × 15-second sub-clips** by the orchestrator. The storyboard agent writes continuity direction (matching eyeline, lighting consistency, body position) between sub-clips.

**Clip Decomposition Strategy:**

```
60-second story → 4 clips of 15s each
  Clip 1: Hook (0-15s) — ARIA establishing shot, intrigue hook line
  Clip 2: Build (15-30s) — tighter MCU, analytical beat, B-roll cue marker
  Clip 3: Payoff (30-45s) — ARIA reaction shot, hot-take wind-up
  Clip 4: Hot Take (45-60s) — ECU (extreme close-up) on ARIA delivery + cut away
```

**ARIA Character Consistency:**
- All clips use identical ARIA character seed description (stored as agent memory)
- Reference-to-Video mode used for clips 2-4, referencing clip 1's first frame
- This ensures consistent hair, wardrobe, skin tone, and set across all segments

**Estimated Generation Time:** ~3 min/clip × ~25 clips = ~75 min total (parallelisable via async queue)

**Cost Estimate Stage 6A:**
- 25 clips × 15 sec × $0.28/sec (1080p) = **~$105 per episode**

---

### Stage 6B: Wan 2.6 — B-Roll Generation

**API:** Alibaba Cloud DashScope (`wan-pro` mode)

**Objective:** Generate thematic B-roll cutaways for each story (8-second clips)

**Wan Prompt Style (cinematic B-roll):**

```python
import dashscope
from dashscope.video.generation import VideoSynthesis

def generate_broll(prompt: str) -> str:
    response = VideoSynthesis.call(
        model='wan-pro',
        prompt=prompt,
        duration=8,
        resolution='720p',   # B-roll in 720p to reduce cost
        aspect_ratio='16:9',
        style='cinematic',
    )
    return response.output.video_url
```

**B-Roll Examples per Story Category:**

| Story Type | Wan B-Roll Prompt Fragment |
|---|---|
| Tech story | "Macro close-up of circuit board, data streams visualised as light, cinematic bokeh, blue-orange palette, slow push-in" |
| Viral social | "Time-lapse of social media notification icons cascading like rain, neon reflections, cyberpunk aesthetic" |
| Science | "Bioluminescent organisms in deep ocean, teal-green light, slow drift, ultra-HD nature documentary style" |
| Politics | "Slow motion confetti falling in empty auditorium, spotlight, desaturated film grain aesthetic" |
| Weird news | "Anthropomorphic elements — mundane objects behaving unexpectedly, Wes Anderson colour palette, still camera" |

**Cost Estimate Stage 6B:** 5 clips × 8 sec × ~$0.09/sec (720p wan-pro) = **~$3.60 per episode**

---

### Stage 7: Post-Production Pipeline

**Runtime:** FFmpeg (subprocess calls from orchestrator)

**Step 7.1 — Segment Assembly:**
```bash
# Concatenate ARIA anchor clips for Story 1
ffmpeg -f concat -safe 0 -i story1_clips.txt \
  -c:v libx264 -c:a aac story1_anchor.mp4
```

**Step 7.2 — B-Roll Insertion (at marked cue points):**
```bash
# Insert B-roll at specified timecode
ffmpeg -i story1_anchor.mp4 -i story1_broll.mp4 \
  -filter_complex "[0:v][1:v]overlay=enable='between(t,20,28)'" \
  story1_with_broll.mp4
```

**Step 7.3 — Episode Assembly:**
All segments concatenated in order: Intro → S1 → S2 → S3 → S4 → S5 → Outro

**Step 7.4 — Lower-Third Captions (Qwen3.7-generated):**
The orchestrator passes script text back to `qwen3.7-max` (instruct mode) to generate:
- Story title cards (animated text overlays via FFmpeg drawtext)
- Headline bullet captions for each story
- Source attribution crawl
- ARIA's "Hot Take" graphic with pulsing animation

**Step 7.5 — Audio Post:**
- Music bed: Free-license electronic news theme (pre-licensed asset library)
- Ducking: FFmpeg sidechain duck music -18dB under ARIA dialogue
- Normalisation: LUFS -14 (YouTube standard), -16 LUFS (X/broadcast)

**Step 7.6 — Colour Grade (FFmpeg LUT):**
```bash
ffmpeg -i assembled_episode.mp4 \
  -vf "lut3d=file=pulse_ai_lut.cube" \
  -c:v libx264 -preset slow -crf 18 \
  final_episode.mp4
```

**Custom LUT:** "Pulse Grade" — boosted teal/orange contrast, enhanced skin tones, deep blacks. Applied globally for consistent brand look.

**Step 7.7 — Thumbnail Generation:**
- Qwen3.7 generates thumbnail concept brief
- HappyHorse Image Generation (single frame) renders ARIA in dramatic pose
- Qwen3.7 generates headline overlay text in YouTube thumbnail format

**Token Budget Stage 7:** ~2,000 tokens (caption writing + thumbnail brief)

---

### Stage 8: Platform Publishing

**YouTube Data API v3:**
```python
def upload_to_youtube(video_path, metadata):
    youtube.videos().insert(
        part="snippet,status",
        body={
            "snippet": {
                "title": metadata["title"],        # Qwen3.7-generated
                "description": metadata["desc"],   # SEO-optimised
                "tags": metadata["tags"],           # 15 viral-relevant tags
                "categoryId": "25",                 # News & Politics
            },
            "status": {"privacyStatus": "public"}
        },
        media_body=MediaFileUpload(video_path)
    ).execute()
```

**X (Twitter) API v2:**
```python
def post_to_x(video_path, episode_title, hot_takes):
    # Upload media
    media_id = client.upload_media(video_path)
    # Post thread
    client.create_tweet(
        text=f"🔴 ARIA is LIVE — {episode_title}\n\n"
             f"This week's Hot Takes:\n"
             + "\n".join([f"• {ht}" for ht in hot_takes[:3]])
             + "\n\n#PulseAI #ViralNews #AIAnchor",
        media_ids=[media_id]
    )
```

**Qwen3.7 Generates:**
- YouTube SEO title (A/B variant)
- YouTube description with timestamps and keywords
- X thread copy (hook tweet + 3-part thread)
- Episode-specific hashtag set

**Token Budget Stage 8:** ~1,500 tokens

---

## 5. ARIA — Character Design Specification

### Visual Identity

| Attribute | Specification |
|---|---|
| **Appearance** | Stylised hyper-realism; not uncanny — deliberately "too perfect" (signals AI identity) |
| **Skin** | Luminous, slightly iridescent undertone — subtle reminder she is synthetic |
| **Eyes** | Silver-blue; slight glow at pupil edge when delivering "hot takes" |
| **Hair** | Structured dark bob with silver highlights — precision cut, never casual |
| **Wardrobe** | Rotating navy/slate/charcoal blazers with statement geometric accessories |
| **Set** | Dark glass news desk, city-at-night bokeh backdrop, Pulse AI animated logo |
| **Lighting** | Broadcast quality: golden key, cool fill, rim light — consistent across all episodes |

### Voice & Performance Personality

```
ARIA's voice characteristics (HappyHorse audio direction):
- Tone: Warm mezzo-soprano with crisp consonants
- Accent: Neutral mid-Atlantic broadcast English
- Pace: 145-160 words per minute (broadcast standard)
- Pauses: Strategic; ARIA holds eye contact during [PAUSE] cues
- Laugh: Barely-suppressed — one per episode, at the "weird" story
- Quirk: Occasionally glances off-camera as if checking with a producer
```

### Signature Verbal Patterns

| Moment | ARIA's Language Pattern |
|---|---|
| Episode open | "If the internet had a fever this week — ARIA has the diagnosis." |
| Story hook | "Here's what nobody is talking about — except everyone." |
| Shocking stat | "Let that sit with you for a moment. [PAUSE: 2s]" |
| Hot Take delivery | "Hot Take:" + slow lean to camera + single devastating sentence |
| Sign-off | "Stay curious. Stay weird. I'm ARIA — and this has been your Pulse." |

---

## 6. Token Budget Summary

| Stage | Model Mode | Est. Tokens | Notes |
|---|---|---|---|
| Stage 2: Editorial Scoring | Thinking | 8,000 | Highest reasoning complexity |
| Stage 4: Script Writing | Thinking | 12,000 | Core creative work |
| Stage 5: Storyboard | Instruct | 5,000 | Structured prompt generation |
| Stage 3: Character Direction | Instruct | 3,000 | |
| Stage 7: Captions + SEO | Instruct | 2,000 | |
| Stage 8: Social Copy | Instruct | 1,500 | |
| Stage 1: Aggregation | Tool calls | 2,000 | |
| **TOTAL** | | **~33,500 tokens** | Well within Track 2 budget |

> **Optimisation Strategy:** Stage 2 processes 50 stories with 150-token summaries (not full articles). Full article text is only passed to Stage 4 for the 5 selected stories. This prevents context bloat at the most expensive reasoning stage.

---

## 7. Agentic Orchestration Design

### Memory Architecture

```python
class PulseAIOrchestrator:
    """Stateful agentic loop managing the full pipeline."""

    def __init__(self):
        self.episode_memory = {
            "aria_character_seed": ARIA_SEED_PROMPT,  # Immutable across episodes
            "selected_stories": [],
            "script": {},
            "storyboard": {},
            "generated_clips": {},
            "episode_metadata": {}
        }
        self.qwen_client = OpenAI(
            api_key=os.environ["DASHSCOPE_API_KEY"],
            base_url="https://dashscope.aliyuncs.com/compatible-mode/v1"
        )

    def run_pipeline(self):
        self.crawl()           # Stage 1
        self.analyse()         # Stage 2
        self.direct()          # Stage 3
        self.write_script()    # Stage 4
        self.storyboard()      # Stage 5
        self.generate_video()  # Stages 6A & 6B (async)
        self.post_produce()    # Stage 7
        self.publish()         # Stage 8
```

### Thinking Mode Toggle

```python
def call_qwen(self, prompt: str, use_thinking: bool = False) -> str:
    messages = [{"role": "user", "content": prompt}]
    params = {
        "model": "qwen3-max",  # or qwen3.7-max on DashScope
        "messages": messages,
        "stream": False
    }
    if use_thinking:
        params["extra_body"] = {"enable_thinking": True}
    
    response = self.qwen_client.chat.completions.create(**params)
    return response.choices[0].message.content
```

### Async Video Generation

```python
import asyncio

async def generate_all_clips(self):
    """Parallelise HappyHorse and Wan generation."""
    tasks = []
    for segment in self.episode_memory["storyboard"]["segments"]:
        # Queue anchor clips
        for sub_clip in segment["sub_clips"]:
            tasks.append(self._generate_anchor_clip_async(sub_clip))
        # Queue B-roll
        if segment.get("b_roll_prompt"):
            tasks.append(self._generate_broll_async(segment))

    results = await asyncio.gather(*tasks, return_exceptions=True)
    # Handle failures with retry logic
    self._handle_generation_results(results)
```

### Error Recovery

```python
def _retry_with_fallback(self, clip_spec: dict, attempt: int = 0) -> str:
    """If HappyHorse 1080p fails, fall back to 720p, then Wan."""
    if attempt == 0:
        return self._generate_happyhorse(clip_spec, resolution="1080p")
    elif attempt == 1:
        return self._generate_happyhorse(clip_spec, resolution="720p")
    else:
        # Final fallback: Wan 2.6 for this segment
        return self._generate_wan(clip_spec)
```

---

## 8. Weekly Production Schedule (Fully Automated)

```
Monday 00:00 UTC  — Cron trigger fires
Monday 00:00-01:30 — Stages 1-5 (Crawl, Analyse, Script, Storyboard)
Monday 01:30-03:00 — Stage 6A/6B (Video generation — async, ~90 min)
Monday 03:00-03:30 — Stage 7 (Post-production — FFmpeg pipeline)
Monday 03:30-04:00 — Stage 8 (Upload + publish to YouTube & X)

Total pipeline time: ~4 hours
Published by:       Monday 04:00 UTC (noon Singapore time — peak Asia viewership)
```

---

## 9. Hackathon Demo Strategy

For the hackathon submission, demonstrate **one complete episode** showing:

1. **Live agent run** (screen recording of orchestrator console logs)
2. **Qwen3.7 Thinking Mode outputs** — show raw reasoning for editorial scoring
3. **Script artifact** — full JSON with performance cues
4. **Storyboard artifact** — HappyHorse prompts with cinematographic language
5. **Final video** — complete ~5.5 min episode with ARIA presenting 5 viral stories
6. **Token efficiency dashboard** — breakdown of tokens per stage vs. allocated budget

### Judging Showcase Moments

- **Narrative Ability:** Play the "Hot Take" delivery from Story 3 — this is the creative peak
- **Multimodal:** Split-screen showing script text → HappyHorse prompt → generated clip → assembled edit
- **Token Budget:** Show that 33,500 total tokens produce a 5.5-minute broadcast-quality video

---

## 10. Infrastructure & Cost Model

| Component | Service | Cost per Episode |
|---|---|---|
| Qwen3.7-Max (33,500 tokens) | DashScope | ~$1.50 |
| HappyHorse 1080p (25 clips × 15s) | fal.ai | ~$105.00 |
| Wan 2.6 B-roll (5 clips × 8s) | DashScope | ~$3.60 |
| Web crawling APIs | Various | ~$2.00 |
| YouTube upload | Free tier | $0 |
| X API (Basic) | X Developer | ~$5/month |
| Compute (cloud VM) | AWS/GCP | ~$1.00 |
| **Total per episode** | | **~$113** |

> **Hackathon context:** Token credits provided by the hackathon cover LLM costs. Video generation costs are the primary investment — quality-justified for a high-production broadcast format.

---

## 11. Open Questions / Design Decisions

> [!IMPORTANT]
> **ARIA's Linguistic Style:** Should ARIA be bilingual (English + Mandarin), leveraging HappyHorse's native multilingual lip-sync? A bilingual format would broaden the X audience significantly and demonstrate HappyHorse's multilingual capability as a technical differentiator.

> [!IMPORTANT]
> **Story Count:** The 5-story format targets ~5.5 minutes. For X (optimal ~90s for video virality), should a short-form "ARIA Highlights" cut (1 story, 90 seconds) be auto-generated as a second output?

> [!NOTE]
> **ARIA's Origin Story:** Including a 30-second "ARIA Origin" mini-episode as the pilot's cold open would establish character lore and make the hackathon submission more compelling as a media property.

> [!TIP]
> **B-Roll Alternative:** If Wan 2.6 API access is unavailable during the hackathon, the orchestrator can fall back to Pexels API (free stock video) + Qwen3.7 to select contextually matching clips — preserving full automation.

---

## 12. Repository Structure

```
pulse-ai/
├── orchestrator/
│   ├── main.py              # Pipeline entry point + cron trigger
│   ├── stages/
│   │   ├── crawl.py         # Stage 1: Web aggregation
│   │   ├── analyse.py       # Stage 2: Qwen3.7 editorial scoring
│   │   ├── direct.py        # Stage 3: Character direction
│   │   ├── script.py        # Stage 4: Script generation
│   │   ├── storyboard.py    # Stage 5: Visual production specs
│   │   ├── generate.py      # Stage 6A+6B: Video generation
│   │   ├── postproduce.py   # Stage 7: FFmpeg pipeline
│   │   └── publish.py       # Stage 8: Platform upload
│   └── memory/
│       ├── aria_seed.py     # ARIA character constant
│       └── episode_state.py # Runtime state management
├── assets/
│   ├── pulse_ai_lut.cube    # Custom colour grade LUT
│   └── music_bed/           # Pre-licensed music assets
├── prompts/
│   ├── editorial_scoring.md # Stage 2 system prompt
│   ├── script_master.md     # Stage 4 system prompt
│   └── storyboard_dir.md    # Stage 5 system prompt
└── README.md
```

---

*Specification Version 1.0 — Pulse AI — May 2026*
*Built for the Global AI Hackathon Series · Track 2: AI Showrunner*
