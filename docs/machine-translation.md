# Pre-made machine translations

Works with no English translation get one made by machine: `<work>~en-mt`. It is aligned with the
work's shown text verse by verse (`loc_scheme: parent`), flagged `meta.machine_translation`
(`mt_engine`, `mt_models`, `mt_date`), and is never the canonical translation: the reader and verse
pages pick a human translation first and show an **MT** pill (engine and date in its tooltip).
Research statistics (`/api/stats`, `/api/query`, `/api/export`) leave machine translations out unless
asked for (`include_mt`).

## Pipeline

```
pramana mt inventory [--out docs/mt-inventory.md]   # what lacks English, words per language, cost of the rest
pramana mt run --batch [--lang sa,pi] [--tier 2] [--budget 100]   # paid; OPENROUTER_API_KEY
pramana mt collect [--wait]                          # store finished batches
pramana mt status                                    # coverage and spend
pramana update --offline --no-build && pramana build # the `mt` adapter stages the cache
```

* **Selection** (`translate.works`/`tier`): a text or commentary with content and no human English
  translation on itself, its recensions, an ancestor text or via a `translates` relation. Tier 1 major
  root texts (catalogue branches of canonical works, commented on, several recensions), 2 other texts
  and major commentaries, 3 other commentaries and the Chinese/Tibetan canons (Taishō 1–1692, Kangyur),
  4 the rest. *Deferred*: page-extracted printed editions and OCR not made by a good engine
  (Gemini 3.1 Pro / Claude Opus, see `ocr-bakeoff.md`); recensions catalogued as texts of their own when
  their work is translated.
* **Context**: each request carries about a page of consecutive segments with the work's title,
  catalogue place, author, date, summary, a glossary of terms and names and the passage just before.
  In batch mode a long work first gets its glossary from one synchronous request over a sample of the
  text; in synchronous mode the glossary grows as the work is translated.
* **Cache** (`raw_data/mt/<engine>/<work>.jsonl`): one line per segment keyed by locator and a hash
  of the source text; `<work>.src.json` lists the segments as last seen. A re-run translates only
  missing or changed segments; the adapter stages only translations of the current source text.
  `ledger.jsonl` records every request's cost. Submitted batches wait in `_batches/`.
* **Provenance**: engine, model and date; CC0 (machine output) — the original keeps its own terms.

## Engine choice

Bake-off (`scripts/mt_bakeoff.py`, [mt-bakeoff.md](mt-bakeoff.md)): 24 passages in 7 languages
with known human translations, 7 engines, judged blind by Claude Fable 5.1 and Gemini 3.1 Pro.

| engine | accuracy (mean over languages) | $ / 1k words (batch) |
|---|---:|---:|
| Claude Opus 5.5 | 8.7 | 0.31 |
| Gemini 3.8 Flash | 8.4 | 0.044 (0.022) |
| Gemini 3.1 Pro | 8.4 | 0.32 |
| Claude Sonnet 5.5 | 8.0 | 0.15 |
| Gemini 3.1 Flash Lite | 7.6 | 0.020 |
| DeepSeek V4 Pro | 7.6 | 0.11 |
| Dharmamitra (MITRA; sa, pi, lzh, bo) | 7.4 | free |

**Gemini 3.8 Flash** for every language: within ~0.3 of Opus everywhere, a seventh of the price, and
half again in batch mode. Dharmamitra is free but drops or invents clauses on Tibetan and harder
Sanskrit (Kaṭha 4.2/10), and takes no work context or glossary; it stays the on-demand engine of the
Translate button. Flash Lite and DeepSeek fail on Prakrit. Opus is the upgrade path for the most-read
works (`--engine openrouter:anthropic/claude-opus-5.5`); where several engines translated a segment the
adapter prefers `translate.PREFERENCE` (Opus > Gemini Pro > Flash).

**Foreign roots** (works outside `in`): a second bake-off ([mt-bakeoff-foreign.md](mt-bakeoff-foreign.md), 17
passages in 10 languages) chose `translate.FOREIGN_ENGINE`: Claude Opus 5.5 for Classical Chinese, Latin, Tibetan,
Greek and Arabic (+0.3–0.8 over Flash), Gemini 3.1 Pro for Avestan, Flash for Persian and cuneiform.  Their prompt
adds `FOREIGN_RULES` (keep the author's own forms of Indian names — Sandrokottos, Palibothra, Tianzhu).  Opus batches
cost ≈ 1.36 × `estimate()`.

## Cost and coverage

About **$0.022 per 1,000 Sanskrit words** in batch mode; Chinese costs more per character. Remaining
work and its cost: [mt-inventory.md](mt-inventory.md).

Round 1 (2026-10-06/07): every tier 1–4 work in Sanskrit (1,546), Pāli (119), Tamil (141), Prakrit and
Apabhraṃśa (6) and the vernaculars (13): 1,825 works, 1.86M segments, **$510.54** (ledger; plus $5.90 for
the bake-off). Left: the Chinese and Tibetan canons (~$713) and the deferred printed-page/OCR texts.

Round 2 (2026-10-10, review R5): ≈ 350 foreign works without a human English translation (Arabic, Persian, Greek,
Latin passages, Chinese, Tibetan, Sumerian/Akkadian, Old Persian, Avestan, papyri), and 617 Indian works that entered
the library after round 1 or lost coverage when W1 re-segmented them (102 tier 1, 451 tier 2, 64 stale):
**$219.70** (Opus 84.55, Flash 134.40, Pro 0.76).  Gaps after collection: 18 foreign and 238 Indian segments.

**Without data/pramana.db** (task worlds): `scripts/mt_mirror.py docs` and `segs --plan works.json` copy the
catalogue and the shown text of the chosen works from the live `/api/sql`; `PRAMANA_MT_DB=<mirror> pramana mt run …`
then works as usual.  Copy segments from a scratch build instead where an adapter's segmentation has changed but
is not live yet — the cache is keyed by locator and source hash, so a re-segmented work loses its translations.
`--budget` stops submitting part-way through a selection: check per-work gaps after `collect`.

## Reproducing `raw_data/mt/`

The cache (~0.9 GB) is paid output — keep it with `raw_data`. To rebuild it from nothing (~$510 at
batch prices, a few hours):

```
pramana mt run --batch --tier 4 --lang sa,pi,ta,apa,pka,pmh,psu,pra,pgd,hi,mr,bra,bn,ml,awa --budget 600
pramana mt collect --wait && pramana mt collect --redo   # store answers; re-read partial ones for free
pramana mt run --batch …same selection… && pramana mt collect --wait   # fills the ~0.2% gaps
pramana stage mt
```

Model output is not deterministic, so a rerun gives different (equally flagged) wording.
