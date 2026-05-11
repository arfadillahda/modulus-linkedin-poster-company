# modulus-linkedin-poster-company

Sibling of [`modulus-linkedin-poster`](https://github.com/arfadillahda/modulus-linkedin-poster).
Generates one LinkedIn post per day for the **Modulus1 Company Page** —
topics rotate over **artificial intelligence**, **sustainability**,
**digitalization**; framing rotates over **TOF / MOF / BOF** (brand
reframe — see `scripts/generate_company.py`). Output is appended to
`data/company_queue.json` and republished as `feed/company.xml`. A
Make.com scenario polls that RSS and posts to the Modulus1 page using
its own LinkedIn OAuth — **separate from the personal repo and its
personal-profile scenario**, by design.

## How it works

1. `scripts/generate_company.py` calls Claude Haiku once per run, picks
   the least-recently-used (topic, stage) cell out of the 9
   combinations, and generates `{title, body}` for one post in the
   Modulus1 brand voice (collective first-person, no corporate filler).
2. On alternating days it also calls Claude Sonnet to produce a 1024x1024
   SVG infographic cover, rasterized to PNG, served from `media/` via
   GitHub Pages.
3. `scripts/build_rss.py` rebuilds `feed/company.xml` from the queue.
4. `generate-company.yml` workflow runs daily at **07:00 UTC** (one hour
   after the personal repo so commit windows don't collide), commits
   the updated queue + feed + media back to the repo, and pushes.
5. Make.com polls the public RSS URL and reposts via the dedicated
   "Modulus1 Org Page" LinkedIn OAuth connection.

## Public RSS URL

```
https://arfadillahda.github.io/modulus-linkedin-poster-company/feed/company.xml
```

(Point your duplicated Make.com scenario's RSS module here. The module
config matches the personal scenario one-for-one — same `enclosure`,
`media:content`, and `content:encoded` semantics.)

## Required secret

- `ANTHROPIC_API_KEY` — Claude API key (uses `claude-haiku-4-5-20251001`
  for text, `claude-sonnet-4-6` for SVG covers).

## Cost

~$0.50–$1.00/month at one post/day with Haiku + alternating Sonnet SVG.

## Voice rules (codified in `SYSTEM` prompt)

- Collective first-person ('we', 'our') — brand voice, not Dam's
  personal voice.
- Direct, technically literate, slightly understated. Senior-engineer
  register.
- Bans: 'unlock', 'leverage', 'game-changer', 'revolutionize',
  'cutting-edge', 'transformative', 'empower', 'synergy',
  'best-in-class', AI-default em-dashes, emoji, hashtag spam.
- Problem -> Solution structure every post:
  - TOF: industry reframe, no product mention.
  - MOF: framework / methodology, named generically.
  - BOF: concrete capability with optional soft showcase of one
    Modulus1-built artifact (e.g. SchemaPin, FormPulse, Strata, Atmos,
    Helio, CV builder), framed around the reader's problem, never
    pitched.
- LinkedIn-native length: 120-220 words.

## Local dry-run

```bash
pip install -r requirements.txt
ANTHROPIC_API_KEY=sk-ant-... python scripts/generate_company.py
python scripts/build_rss.py
cat feed/company.xml
```

## File map

| Path                          | Purpose                                  |
|-------------------------------|------------------------------------------|
| `scripts/generate_company.py` | Claude → text + SVG infographic + queue  |
| `scripts/build_rss.py`        | Queue JSON → `feed/company.xml`          |
| `data/company_queue.json`     | Append-only post log (capped at 200)     |
| `feed/company.xml`            | Public RSS for Make.com to poll          |
| `media/<id>.png`              | Rasterized SVG covers, served via Pages  |
| `media/<id>.svg`              | Source SVG (debug / regen reference)     |
