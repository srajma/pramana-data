# Round 2026-10h — summary (integrator #111): first and second releases

The user's message after the 10g deploy: build in Museums of India, NMMA, the V&A and the British Museum; find Yardi's
1986 Mahābhārata book; English for the Chinese and Tibetan canons (existing translations first); make the library
comprehensive against the great catalogues; a new section for Islamic-period and later sources (`in.darkages`); and a
list of features and bugs. Later in the round: the live server's memory, lost fractions in report measurements,
classic secondary sources (`ins.`), visitor analytics.

This release ships the workstreams that have landed; R3 (comprehensiveness), S1 (secondary sources) and A8 (analytics)
follow in the next.

| id | what | report |
|---|---|---|
| U7 | "me" as a claim source (`user:<name>`); the model named on every OCR/MT action and on hover; English MT always a default companion where no human English exists; tables of contents in order (Kāmasūtra and 78 more texts, `pramana check --toc`) | [U7.md](U7.md) |
| M3 | Yardi's strata for the Śāntiparvan (172 of 353 adhyāyas, from his 2001 *Epilogue*), his own later corrections applied, the Harivaṃśa's style; the 1986 book is not online | [M3.md](M3.md) |
| F4 | lost fractions in report measurements: a clean-up bug dropped TeX fractions (5,930 passages fixed, ids stable), 2,748 fraction-dense pages re-read with a referee (1,303 corrections), artefact sizes implausible for their type 1,520 → 116; $58.92 | [F4.md](F4.md) |
| D1 | `in.darkages` (Literature): 1,641 entries from Storey, Tessitori and Ganda Singh; Peshwa Daftar (44 vols, 10,004 documents) and Rajwade (11 vols, 3,078) with dates and summaries; Elliot & Dowson complete; 20 public-domain translations (Āʾīn-i Akbarī, Akbarnāma, Badāʾūnī, Firishta, Jahāngīr, Bernier, Tavernier, Manucci …); ≈ $105 | [D1.md](D1.md), [D1-survey.md](D1-survey.md) |
| T2 | English for the canons: 84000 (432 Tibetan texts), BDK (15), SuttaCentral Āgamas (394), Lotsawa House (86), Lapis Lazuli (11) shown in full; copyrighted English linked; machine translation for every canon text still without English and 299 major non-canon texts; $727.64 | [T2.md](T2.md) |
| F3 | museum catalogues: Museums of India archaeology galleries (34,761, all pictured), the national museums' photographs on (33,761 records), British Museum (23,260 objects + 25,844 coins, 2014 open data), V&A (5,180); objects → inscriptions, manuscripts → works; NMMA unreachable from every network tried (ASI permission is the route) | [F3.md](F3.md) |
| O1 | live API workers' memory (deployed separately as PR #96): image work out of the request workers, SQLite heap capped, 16 threads, allocator tuned, graceful recycle | [O1.md](O1.md) |

Release fixes by the integrator: Lotsawa House translations filed under the most specific of their listed matches
(paths had " | " in them); an edition that fills a placeholder made by its own machine translation takes the edition's
title; the Buddhacarita's Derge Tengyur edition catalogued.

## The release (uploaded 2026-10-23)

R2: the 10g files archived as `archive/pramana-2026-10-21.db`, `archive/words-2026-10-21.db`; `deploy/data.manifest`
updated. pramana.db 25.69 GB, words.db 3.96 GB, lexicon.db unchanged.

| | 10g (live) | 10h |
|---|---:|---:|
| documents | 348,385 | **446,125** (Literature 77,673 · Physical 365,468 · Foreign 2,984) |
| artefact records / museum coin specimens | 100,972 / — | **164,044 / 48,773** |
| English translations (documents) | — | **43,962** |
| passages | 13.74 M | **14.39 M** |
| words | 514.9 M | **670.3 M** |
| `in.darkages` | — | 1,641 entries; 126 readable (35 in the original, 91 through translations) |

Every "must be zero" catalogue check 0 (after the three release fixes above); full test suite and web check clean.

## After the deploy

* `python -m pramana.api.imgproxy warm` (+ `--epigraphy`) on the server (museum photographs come on first view).
* The V&A's records must be refreshed within four weeks of 2026-10-21 (its terms): the next release refetches them.
* O1's watch script (`scripts/o1/watch.sh`) for an hour of worker memory, when the server login is granted.

## Second release (uploaded 2026-10-27)

Release builder task_mvoxr7so5f02f75884 on 8 CPU / 16 GB, from the live first release; no paid calls (every OCR,
front-check and locate verdict was cached).

| id | what | report |
|---|---|---|
| R3 + R4 | comprehensiveness: the NCC and Aufrecht's left-outs catalogued (125,329 works); redlinks filled from archive.org editions (R3 2,008, R4 2,663) and sanskritdocuments | [R3.md](R3.md), [R4.md](R4.md), [R4-filled.tsv](R4-filled.tsv) |
| S1 | classic secondary sources under `ins.` "Studies" (329 books, 67.1 M words, 16,319 figures in place); their citations linked both ways ("Discussed in", 711,407 links) | [S1.md](S1.md) |
| C1 | commentaries linked verse by verse (align job `commentaries`, `pramana check --commentaries`); new adapter `wsbhasya` (Sāyaṇa, Uvaṭa, Mahīdhara, the Gītā commentators from Sanskrit Wikisource, Pandit's Atharvaveda) | [C1.md](C1.md) |
| K8 | modern compilations: `in.lit.subhasita.subhasita.adhunika` (Subhāṣitaratnabhāṇḍāgāra 12,750 verses, Mahāsubhāṣitasaṃgraha, Böhtlingk's Indische Sprüche with German and notes, Jīvānanda's Kāvyasaṃgraha); verses linked to their sources (`quotes` 2,718 → 16,146) | [K8.md](K8.md) |
| F3 | V&A records refetched 2026-10-26 (their terms: refresh within four weeks); 5,180 objects | [F3.md](F3.md) |
| V1, M4, A8 | code only (search-engine pages, manuscript OCR choice, analytics); in the branch, no data of their own | [V1.md](V1.md), [M4.md](M4.md), [A8.md](A8.md) |

Release fixes by the builder: `segsplit` treated an 11,835-digit OCR run as a locator and crashed `int()` (a number
part longer than 7 digits is no locator now); the reader skipped a commentary passage that only repeats the verse
without showing the comment that follows as its parts, so Śaṅkara was no longer shown on the Taittirīya (C1
regression, `test_api_linked`); `pramana release --upload --status` uploaded instead of reporting (it now reports).

**R2:** the first release's files archived as `archive/pramana-2026-10-23.db` and `archive/words-2026-10-23.db`;
`deploy/data.manifest` updated. **pramana.db 29.92 GB** (29,923,176,448 B; was 25.69), **words.db 6.01 GB**
(6,013,898,752 B; was 3.96; rewritten, not sparse), lexicon.db unchanged.

Before/after with the same queries on both libraries (`doc`, `segment`, `relation`):

| | 10h first release (live) | 10h second release |
|---|---:|---:|
| documents | 446,125 | **571,608** (Literature 202,799 · Physical 365,468 · Foreign 2,984 · Studies 357) |
| Literature works with text (kind text, available or partial) | 13,566 | **16,999** |
| Literature redlinks | 22,781 | 128,596 (the NCC and Aufrecht catalogued) |
| passages (segments with text) | 13.68 M | **18.07 M** |
| words | 670.3 M | **803.0 M** |
| English translations (documents) | 43,288 | 43,166 (the build drops machine translations of texts that moved: 321) |
| commentaries with text / `comments_on` links | 1,671 / 514,930 | **2,821 / 985,788** |
| studies (`ins.`) / "Discussed in" links | — | **329 books / 711,407** |
| artefacts (V&A) | 164,044 (5,180) | 164,044 (5,180, refreshed) |

Checks: every "must be zero" catalogue check 0. Link check: 4,514 dangling locator links (passages-sa 4,157,
anthologies 172, fragments 100, commentaries 77), and 20 link targets missing, all in K8's
`catalog/links/anthologies.tsv`: Jaina and other editions R4's restage named differently (e.g.
`padmapurana*pannalal1999` is `*vrajlal1999` now). `pramana check --commentaries`: of 2,821 commentaries with text,
1,620 linked, 141 partly, 913 with a stated reason, **147 unlinked**, all new archive.org editions from R3/R4
(`data/release/commentaries.txt`, not blocking). Full test suite: 146 files, 1,286 passed. Four files failed in the
first pass and pass now: `test_api*` ran while an aborted build held the database (rerun 13 passed),
`test_words` passed on rerun, `test_api_linked` was the reader regression above. `npm run build` and
`svelte-check` clean (0 errors, 0 warnings). Spot-checks on the built library, served locally: Ṛgveda 1.1.1 with Sāyaṇa;
Gītā 2.47 with 49 commentaries; Subhāṣitaratnabhāṇḍāgāra 8.412 → Meghadūta (`quotes`) and the Meghadūta's "quoted in";
Fergusson 1876 ch. 12 with 28 figures and its citations, the Meghadūta discussed in 87 studies; R4 fills
(Dattātreyasaṃhitā, Nyāyasindhu, Vihagendrasaṃhitā, Pavanavijaya) with text; V&A IS.137-1954 with photographs.

### After the deploy

* `docker exec pramana-web pramana indexnow` (V1: after every deploy; the first run only records).
* `python -m pramana.api.imgproxy warm` (+ `--epigraphy`) on the server; the studies' figures warm page by page on
  first view.
* V1's crawler check (`python scripts/v1/visibility.py crawler`) and the JSON-LD re-validation (V1.md, After deploy).
* A8's analytics: one new Python package in the image (`maxminddb`); see A8.md, Deploy.
* The server needs ≈ 6.3 GB more disk for the new files (pramana.db +4.2 GB, words.db +2.1 GB).
* Done 2026-10-28: [after-deploy-2.md](after-deploy-2.md) — deploy, IndexNow, warm-ups, crawler and JSON-LD checks,
  analytics, the memory watch (FAIL: workers recycle every few minutes under crawler load), spot-checks; fixes for the
  artefact index (500), analytics growth, HEAD and the anthology source link; two data issues for the next release.

### Next release

* Regenerate K8's `catalog/links/anthologies.tsv` against this library (`python -m pramana.anthology match <book>
  data/pramana.db` for each book, then `links`; local, free): 20 targets renamed by R4's restage.
* Link or explain the 147 unlinked archive.org commentaries (`catalog/align.yaml`, `catalog/commentaries.yaml`).
* The V&A must be refreshed again by 2026-11-23.
