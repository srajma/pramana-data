# Round 2026-10i release (K9 catalogue order, K9b, L3, W6, T3, C2, S2)

Release sub-task of the 10i integrator (#137, task_mvsitnou5bac9c9f45); run on 8 CPU / 16 GB, 2026-10-29 → 31.
Built from the live 10h library (`deploy/fetch-data.sh`) with every sub-task's delivered stage; uploaded to R2 with
`pramana release --upload` (the live files archived in R2 as `archive/pramana-2026-10-27.db`, `archive/words-2026-10-27.db`,
`archive/lexicon-2026-10-18.db`, `archive/semantic-2026-10-28-2.idx`, `archive/semantic-words-2026-10-28.idx`), every
upload read back and its sha256 compared; `deploy/data.manifest` committed.
Nothing deployed: the deploy happens when this branch lands on master.

## What ships

| | |
|---|---|
| **K9** | the catalogue order: ≈ 145k moves (`catalog/moved/k9.tsv`), old paths kept as aliases, code paths rewritten |
| **K9b** | catalogue hygiene (identical texts detached, website-named recensions renamed, placeholder works filed) |
| integrator | untitled ghost nodes folded into their works; K9 round 14 (21 duplicates merged) |
| **L3** | the Vasudevahiṃḍī Part I by lambha; the Madhvavijaya (Wikisource) and Vedāṅgatīrtha's Padārthadīpikā; later śāstra from Sanskrit Wikisource; adapter `gaudiya`; archive.org volumes; 8 new default recensions |
| **W6** | adapters `andhrabharati` (classical Telugu) and `vachanasanchaya` (21,644 Kannada vacanas) |
| **T3** | adapter `insiders` (translations by insiders of the traditions), the GRETIL Kaṭha refrain fix (vallīs 4–6 renumbered), new default English translations |
| **C2** | study citations to the exact passage (numbering concordances, edition pages, named parts, footnote tables) |
| **S2** | regex, lemma, root and grammar in the search box (code + two indexes and `form_alt` in words.db) |
| on master | U8, U9, V2, O2 (code); the Vedic-accent fix (anusvāra before the tone mark, 82f86d6c) |

Not in this release, by the integrator's decision: **C3** (its catalogue merges and `scans.yaml` need a restage; next
release).

## Numbers (same queries as the 10h report)

| | 10h second release (live) | 10i |
|---|---:|---:|
| documents | 571,608 | **573,529** (Literature 204,728 · Physical 365,466 · Foreign 2,975 · Studies 356) |
| path aliases (old URLs that resolve) | — | **1,212,234** |
| Literature works with text | 16,999 | 16,530 |
| Literature redlinks | 128,596 | 153,798 |
| passages (segments with text) | 18.07 M | 17.37 M |
| words | 803.0 M | 787.2 M |
| English translations (documents) | 43,166 | **43,806** |
| commentaries with text / `comments_on` links | 2,821 / 985,788 | 2,122 / 919,139 |
| studies / "Discussed in" links | 329 / 711,407 | 329 / **745,446** |

The fall in passages, words, works and commentaries with text is the deduplication, not lost text: K9b found 676
groups of byte-identical texts in the 10h library (one archive.org book accepted for several entries) — 899 extra
copies, **610,636 passages** (508 texts, 344 commentaries, 34 recensions, 13 translations) now shown once; this release
detached 50 more groups (34,142 passages, below). K9's merges fold further duplicate entries into one node each; the
redlinks grew with K9's 2,768 new categories and R3's works now filed in them (K9 + L3 catalogued more expected works).

Files: **pramana.db 36.00 GB** (35,996,372,992 B; was 29.92), **words.db 5.10 GB** (was 6.01; rebuilt from scratch,
compact), lexicon.db 274 MB (rebuilt: K9 moved the curated analyses' paths), semantic.idx 7.36 GB (12,267,344
passages), semantic-words.idx 204 MB. The server needs ≈ 4.8 GB more disk.

## Steps and timings

| step | time | notes |
|---|---:|---|
| restage (68 adapters) | 3 h 53 min | archiveorg 2 h 33 min with 3 stage workers, the rest 81 min; gretil first, then the Kaṭha base for insiders regenerated from the new gretil stage (T3's `base` command reads the live site, which still had the old numbering; the old base matched the old stage exactly), then insiders; V&A records refreshed (`python -m pramana.artefact_open vam`, due by 2026-11-23) |
| fetch the live library | 59 min | 44 GB |
| first build | **killed twice** | out of memory as the search index started (251 MB of 16 GB left, 2 h of work rolled back each time — one transaction), with 8 and with 3 workers; fixed in code (below), then 3 h 58 min |
| study citations resolved, K8 anthology links | 13 + 20 min | |
| pass 1: stage, build, align | 5 h 42 min | stage 38 min, build 3 h 08, align 1 h 42 (translations $7.86); stopped by `check_catalog --strict` (fixed, below) |
| pass 2: build … semantic | 15 h 03 min | build 3 h 40, align 28 min, checks 7 min, **words 8 h 48 min** (from scratch: lexicon.db changed), cache 27 min, semantic 38 min (**$18.24**: 455,021 new or changed passages embedded, 11.8 M reused) |
| S2's words.db additions | 1 min | `FINAL_INDEXES` + `form_readings`: `form_alt` 1,928,851 rows |
| pass 3 (Mahāniddesa pages, below) | 3 h 10 min + 1 h 13 min | resolve 13 min, stage 10 min, build 2 h 17, align 15 min, checks 9 min; words updated (21 min, 0 documents changed), cache 32 min, semantic 16 min ($0) |
| upload | 1 h 25 min | 5 files archived, uploaded, read back and verified (sha256); pramana.db 43 min |

Paid calls: semantic $18.24, alignment (translations) $7.86, K9b's judge on 50 groups $0.10 — **$26.20**.

### The memory fix (commit 1e7de341)

`searchindex.update_segments` held every changed passage's text in Python lists (K9's moves: 6.5 M entries to delete,
5.9 M to index) and forked its workers after that; 9 GB in the main process beside three workers forked from 7 GB.
Now the pool forks first and the entries go through temp tables read a page at a time: 47 min, ≥ 9 GB free throughout.
The killed build had also left its three workers behind (orphans holding 7 GB each): pool workers now die with the
build (`PR_SET_PDEATHSIG`). Tests: `tests/test_searchindex_bounded.py` (the streamed update equals a fresh index;
workers die with a SIGKILLed parent). `docs/release.md` has the account limit (16 cores in all: a 16-CPU resize stays
pending) and `PRAMANA_WORKERS=3` for 8 CPU / 16 GB.

### Fixes the checks needed (each with a test where code changed)

* **identical-text 50** (must be zero): archive.org books accepted for two entries after K9b ran. K9b's own judge on
  just these groups (`scripts/k9b/release_identical.py`, scans only, no merges; $0.10): the others' scans
  `status: duplicate`, three books that are none of their entries `rejected` (the Sanskrit Pañcīkaraṇa filed under
  Jñāneśvara's Marathi work, Bhāṣāpariccheda). Simulated on the staged records: every group keeps at most one text.
  The build honours it without restaging archiveorg (`stale_filter`); the stage's fingerprint was adopted
  (`pramana update --adopt archiveorg`) instead of a 2.5 h restage.
* **site-named 2**: empty "Durveka-sarit" / "Tippanaka-sarit" made by machine translations at pre-K9 paths: the mt
  adapter followed exact moves only, not a moved ancestor (K9 moved the Nyāyabindu). Now `moved.follow`; SARIT's
  Dharmottarapradīpa MT on its edition `durveka*alt` (`tests/test_f1_imperfections.py`).
* **placeholder-work**: Raudrī's scan under an R3 path the catalogue lost → `catalog/moved/release-10i.tsv`. The other
  15 are K9b's open episcans inscriptions (K9b §4): the release ran `--no-strict` only after a script confirmed they
  were the only failures.
* **`check --commentaries`: 341 unlinked** → 0: K9 filed ≈ 300 works as commentaries under their bases (archive.org
  scans; Esukhia's Tengyur texts, divided by folio); this release's align job found too little evidence for them;
  reasons regenerated with `scripts/c1/reasons.py`.
* **J. C. Jain's Vasudevahiṃḍī pages** (5 of 40 resolved): the extraction left the page in the quote ("Vasu., p. 20")
  or gave a range ("235-240") → their first page, for scanned editions cited by page (`tests/test_studies_s1.py`).
* **Moti Chandra's Mahāniddesa** (all to the work): "Vol. I, pp. 154-155" stands after the editors' names, beyond the
  40-letter window the resolver looked at → a page printed on the citation's own line counts, ranges included
  (pass 3; `tests/test_studies_s1.py`).
* `tests/test_api_w3.py` named two pre-K9 paths (Dīrghāgama, Lotus) — updated.

## Checks on the built library

| check | result |
|---|---|
| `pramana check --order` | every must-be-zero line **0** (453 large categories, 15 commentaries listed as works: review) |
| `scripts/check_catalog.py --strict` | **0** except `placeholder-work` 15 — the episcans inscriptions K9b §4 leaves open (Kolagallu, Kuruvatti, Kāḷahasti, Tripurāntakam …) |
| `pramana check --commentaries` | **0** unlinked (2,720 with text: 1,445 linked, 138 partly, 1,137 with a stated reason) |
| `scripts/r1/check_old_urls.py` | 35 old paths: **0 problems** |
| K9's moves (`scripts/release/check_moved.py`, 2 × 200 random lines of `catalog/moved/k9.tsv`) | **400 of 400** redirect (301) to the new path and the API answers with that document |
| live 10h paths | 571,547 of 571,608 resolve (document, alias, or under an aliased ancestor); **61 do not** — mostly empty parents K9b emptied (`in.vedanga.vyakarana.paninisutra`, `in.veda.svadhyaya.chandogya`), duplicate copies K9b removed, malformed `~en-mt*mt` paths: they 404 after the deploy (left, below) |
| link check | 6,561 dangling locator links (passages-sa, mitra, fragments), 4 missing documents (`brhadyatra*pingree`, `durveka-3`, `tantravarttika*subbasastri!nyayasudha`, `asvalayana*vidyaratna`: link files naming detached or renamed editions) |
| study citations (`pramana check --citations`) | text citations 1,196,663: passage 402,702 · page 25,866 · section 37,735 · work 581,903 · unresolved 148,444 |

C2's table beside this release's levels (C2 measured on the live 10h library):

| level | C2 before | C2 after | **10i release** |
|---|---:|---:|---:|
| passage | 367,118 | 397,957 | **402,702** |
| printed page | 20,014 | 26,557 | **25,866** |
| section | 35,018 | 37,248 | **37,735** |
| linked below the work | 422,150 | 461,762 | **466,303** |
| work only | 624,199 | 587,333 | **581,903** |

Moti Chandra 1977 (`ins.economy.chandra-1977`): footnote c11.s2.1 (passage `c11.s1.53`) links the Vasudevahiṃḍī pp.
177, 187, 188, 191, 192 to passages 4.p37, 5.p9, 5.p10, 5.p13, 5.p14 (level page), the Mahāniddesa I 154–155 and II
414–415 to `mahaniddesa*pts` 153.10 (the passage running onto p. 154), 414 and 414.2, the Milindapañha to Trenckner's
pages; `/api/discussed/` of the Vasudevahiṃḍī (11 studies, 74 citations) and of the Mahāniddesa (24 studies) list him.
The Vasudevahiṃḍī's page citations: Moti Chandra 5 of 15 resolve to pages (the other 10 print no page), J. C. Jain
**24 of 40** (was 5; the other 16 print none, or "P 111").

## Spot-checks (local `pramana serve`, `PRAMANA_USER_DB` outside `data/`)

Screenshots at 1280 and 390 in [img/release/](img/release/) (`scripts/release/shots.mjs`: Chrome's DevTools protocol,
no Playwright):

| | 1280 | 390 |
|---|---|---|
| in.veda: 5 divisions, Other works last | ![](img/release/veda-1280.png) | ![](img/release/veda-390.png) |
| Ṛgveda 1.1 with Mādhava's Vedabhāṣya (`!vedabhasya`, OCR) | ![](img/release/rgveda-madhava-1280.png) | ![](img/release/rgveda-madhava-390.png) |
| Maitrāyaṇī recension | ![](img/release/maitrayani-1280.png) | ![](img/release/maitrayani-390.png) |
| Svādhyāya › Sūkta › Kāmya (Vāstusūkta) | ![](img/release/svadhyaya-kamya-1280.png) | ![](img/release/svadhyaya-kamya-390.png) |
| Madhvavijaya sarga 1 with Vedāṅgatīrtha's Padārthadīpikā | ![](img/release/madhvavijaya-1-1280.png) | ![](img/release/madhvavijaya-1-390.png) |
| Vasudevahiṃḍī 3.p27 | ![](img/release/vasudevahindi-3.p27-1280.png) | ![](img/release/vasudevahindi-3.p27-390.png) |
| Manucaritramu | ![](img/release/manucaritramu-1280.png) | ![](img/release/manucaritramu-390.png) |
| Akkamahādēvi's vacanas | ![](img/release/akkamahadevi-1280.png) | ![](img/release/akkamahadevi-390.png) |
| Kaṭha 5.1 with Gambhirananda | ![](img/release/katha-5.1-1280.png) | ![](img/release/katha-5.1-390.png) |
| Carakasaṃhitā, Versions › Commentaries (58: Āyurvedadīpikā, Jalpakalpataru, Carakasaṃhitāṭīkā, Carakatattvapradīpikā with text) | ![](img/release/caraka-1280.png) | ![](img/release/caraka-390.png) |
| S2 `'agni' case:loc in:in.veda.samhita.rg` (138 passages, forms marked) | ![](img/release/s2-agni-loc-1280.png) | ![](img/release/s2-agni-loc-390.png) |
| S2 `/agn[iy]\w*/ in:in.veda.samhita.rg` | ![](img/release/s2-regex-1280.png) | ![](img/release/s2-regex-390.png) |
| Moti Chandra's footnote with its links | ![](img/release/motichandra-c11-1280.png) | ![](img/release/motichandra-c11-390.png) |
| Vasudevahiṃḍī, About › Discussed in | ![](img/release/vasudevahindi-discussed-1280.png) | ![](img/release/vasudevahindi-discussed-390.png) |
| Ṛgveda 3.1 in Devanāgarī (तवसं॒, no dotted circles) | ![](img/release/rgveda-3.1-accents-1280.png) | ![](img/release/rgveda-3.1-accents-390.png) |

## Tests

Full suite file by file on the release branch (163 files): **1,475 passed, 0 failed, 1 skipped** (rerun after the
fixes below). The first run (with a 25-min limit per file, during the upload) failed in three files, none from the data:
`test_api_w3` named two pre-K9 paths (updated); `test_sections::test_api_old_paths` got another test library's
`/api/index` from O2's page cache, which tests left in `data/pagecache` — `tests/conftest.py` now puts it in a temp
directory (as `user.db`), `test_o2_pagecache` keeps its own; `test_words::test_real_index_pages` took > 10 s while
the upload ran (6.6 s alone); `test_incremental` hit the 25-min limit (passes; 40 min). Web: `npm run check` 0 errors,
0 warnings (390 files); `npx vite build` clean. S2: `tests/test_s2_wordq.py` 35 passed.

## How S2 joined after the build (for the next release)

S2 was cherry-picked (its own three commits; the integrator branch also carries C3) after words.db was built. Its code
changes neither database: the build imports only `scope_clause` from `pramana/api/search.py` (unchanged), and S2's
`wordbuild` change (curated analyses under moved paths) finds nothing here — all 15 curated paths are documents. The
release's input hashes for build and words were therefore re-recorded (`set_db_info`), not rebuilt; the search cache
was recomputed with S2's code. Pass 3's rebuild rewrote one text's segments (the Rāmacaritamānasa, Hindi: not in
words.db), so words.db was updated by the pre-S2 `wordbuild.py` (its fingerprint; S2's would have started afresh,
8.8 h) — 0 documents changed — and S2's `form_alt` + indexes re-applied. **The next words step starts from scratch**
(words.db's fingerprint carries the pre-S2 code).

## What is left

* **The 15 episcans placeholders** (K9b §4): episcans files further editions under the previous build's inscription.
* **61 live paths without a target** (`docs/work-2026-10i/release-404.txt`): ghost parents and removed duplicates —
  a `catalog/moved` line each (and a rebuild) would keep them working.
* **304 accepted, fully OCRed commentary scans are still redlinks**: none is staged (`_entry_records` returns nothing
  on this branch); the integrator's C3 pass is on it.
* **Curation**: "Cakraṭīkā" is a redlink beside Cakrapāṇi's Āyurvedadīpikā (the same work); K9b's 509 "same work?"
  pairs; 4 link-file targets missing (link check).
* `check_old_urls.py --db` (a random sample of verses) scans the 30 GB segment table and did not finish in 15 min.
* The R4 commentaries: done (the 147 of 10h are linked or have reasons).

## After the deploy

1. `docker exec pramana-web pramana indexnow` — ≈ 146k old paths now redirect (K9).
2. `python -m pramana.api.imgproxy warm` (+ `--epigraphy`, `--studies`) on the server.
3. The O2 page cache starts a new generation by itself (new database files and code); warm it:
   `docker exec -d pramana-web sh -c "python -m pramana.api.pagecache warm > /tmp/warm.log 2>&1"`, then
   `pagecache status`.
4. Check on the live site: `/search?q='agni' case:loc in:in.veda.samhita.rg`, `/search?q=lemma:gam tense:aor`,
   `/search?q=/agn[iy]\w*/ in:in.veda.samhita.rg`, `/search/advanced` (S2); an old K9 path redirects
   (`/in.veda.svadhyaya.vastotpattisukta` → `…sukta.kamya.vastotpattisukta`); Moti Chandra's footnote.
5. Disk: ≈ 4.8 GB more on the server (pramana.db +6.1 GB, words.db −0.9 GB, semantic.idx −0.4 GB).
