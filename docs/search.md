# The search language

One line of words and filters that works in every search box, in links (`/search?q=…`) and in the API
(`/api/search?q=…`). The advanced search form (`/search/advanced`, linked under the search bar on the front page and
from every results page) writes it; editing the line fills the form back.

```
muṇḍa-jaṭila (in:in.niti.dharma OR in:in.niti.artha OR in.bauddha) date:[,200]:traditional -type:mt
```

| Written | Means |
|---|---|
| `how do I stop worrying about how things turn out` | by meaning (below); a short query — a name, a term, a word of the library's languages — also as words |
| `'kṛṣṇa'`, `'dharma kṣetra'` | the words only, not by meaning: any script or spelling, with their forms, other names and Sanskrit/Pāli words for English ones |
| `"evaṃ mayā śrutam"` | exactly as written: no other forms or names (spaces do not count in Sanskrit, Pāli, Prakrit, Tamil) |
| `'kṛṣṇa' sort:date` | works oldest first (earliest mentions) instead of best first |
| `(kapota OR pārāvata)` | either word; alone, each is searched with its spellings, forms and expansions |
| `-haṃsa`, `-"pigeon house"` | passages without it |
| `in:in.niti.dharma`, `in.bauddha` | within a category, work or passage (works listed there by an alias count); one `in:` that is a work lists its passages, a locator works too (`in:in.niti.artha.kautilya.13.4`) |
| `-in:inp` | not within |
| `type:text,commentary` | text (any edition) · recension · commentary · translation · mt · object |
| `lang:sa,pi` | language codes (`/api/search/options` lists them) |
| `date:[-500,200]`, `date:[,200]`, `date:500bce..200ce`, `date:<200` | the date window overlaps these years |
| `date:[-500,0]:within` | only dates wholly inside the range |
| `date:[,200]:traditional` | … in a chronology: `traditional` (sources of kind tradition), `scholarly` (any scholar), `any` (any account), or a source id (`date:[,0]:brockington1998`). None: the library's own dating |
| `author:kalidasa`, `author:"Kālidāsa"` | a person id or name; a work's recensions and translations count, its commentaries do not |
| `translator:` | likewise |
| `status:available,partial` | available · partial · redlink · lost (mostly for lists) |
| `quality:clean` / `quality:ocr` | leave out / only text read by OCR |
| `xlang:off`, `names:off`, `forms:off`, `apart:on` | the search options (Sanskrit & Pāli words for English ones, other names, case forms, words apart) |
| `-` before a filter or `(…)` | negates it; `OR` (or `\|`) joins alternatives inside `( )`, one level |
| `lemma:gam`, `lemma:"agni"` | every form of the lemma, in any script or spelling (and only its forms: no other words); each passage marks the form found |
| `lemma:gamana#2`, `lemma:gamana²` | one homonym, numbered as on the word pages |
| `root:gam` | the root and every lemma derived from it (gata, gamana, āgama …, where the dictionaries say so) |
| `'agni' case:loc`, `lemma:gam tense:aor`, `'deva' num:du` | forms with these grammatical features: a feature qualifies the word or lemma before it |
| `person:1 mood:opt in:in.veda.samhita.rg` | no word: every form with the features, within the scope |
| `case:loc,ins`, `-num:sg` | any of the values · not this one |
| `gram:gold` / `gram:auto` | curated analyses only (Ṛgveda, Mahābhārata, Rāmāyaṇa and four other texts) / automatic ones only |
| `/agn[iy]\w*/`, `re:"agnir vai"` | a regular expression over the normalised IAST (lower case, no accents or punctuation); a word typed in an Indic script is transliterated |
| `/agním/raw` | … over the text as stored (accents, punctuation, its own script) |

* **Filters without words** list the matching texts, readable ones first, then oldest first in the chronology asked for (`mode: catalog`):
  `in:in.niti.dharma date:[,200]:traditional`. Recensions and machine translations only when their type is asked for.
* **Versions are dated by their work**: an edition or a translation (an English translation of 1891) counts with the
  date of the text it renders; a commentary has its own date.
* **Chronologies.** A text without a claim of the chronology takes its nearest ancestor's (a Vedic text the Veda's
  traditional date). Passages with dates of their own — the epics' layers, claims made for a section or verse range,
  and the library's per-verse dating (`segment.date_s/e`) — are judged passage by passage.
* **The Studies** (`ins.`, modern scholarship) are searched with the sources since 2026-11-04 and ranked after them
  when they match as well (`STUDY_PENALTY` a passage, `STUDY_WORK_PENALTY` a work, in `pramana/search/engine.py`);
  the Corpus facet row narrows to them, `-in:ins` leaves them out, `in:ins.…` searches one study.
* Mistakes are notes, not errors (`notes` in the answer): an unknown type is dropped, `foo:bar` is a word.
* Code: `pramana/search/syntax.py` (parse, write back), `pramana/search/advanced.py` (filters → the engine's document
  predicate, passage dates, the catalogue), `pramana/search/wordq.py` (lemmas, roots, patterns, grammar),
  `web/src/lib/searchquery.ts` (the form ⇄ the line). API:
  `/api/search/parse?q=` (items, canonical line, labels, notes), `/api/search/options` (languages, chronologies).
* **Charts.** `/api/search/chart` (same parameters as `/api/search`) gives the matches by century: each text's
  passages spread over its work's dates (in the chronology of the query's date filter), with the words of the texts
  searched in each century for rates per 1,000 words. The results page shows it with the languages and branches
  (the facets) under "chart" (`&chart=1` in the URL).

## Words by lemma, root, pattern and grammar

`pramana/search/wordq.py` (evaluation), `pramana/search/gram.py` (the analyses), `syntax.py` (the line).

```
'agni' case:loc in:in.veda.samhita.rg        the locatives of agni in the Ṛgveda
lemma:gam tense:aor                          aorists of gam, anywhere
root:gam in:in.veda.samhita.rg               every word of the root gam in the Ṛgveda
'deva' num:du date:[,-500] sort:date         duals of deva in texts before 500 BCE, oldest first
(lemma:gam OR lemma:yā) tense:aor in:in.veda either verb in the aorist
lemma:vac person:1 -num:sg                   1st-person dual and plural forms of vac
person:1 mood:opt in:in.veda.samhita.rg      every 1st-person optative of the Ṛgveda
tense:aor gram:gold in:in.lit.mahabharata    aorists of the Mahābhārata, curated analyses only
/agn[iy]\w*/ in:in.veda.samhita.rg           a pattern; /…/raw on the text as stored
/^atha\b/ in:in.veda.upanisad                passages opening with atha
```

* **What a feature qualifies.** The nearest word (`'agni'`, or a plain word) or `lemma:`/`root:` before it
  (`'agni' case:loc 'indra' case:nom`); a feature before any of them qualifies the first; with none at all the
  features stand alone and every form that has them is found. Features never qualify a pattern. A word without a
  feature is searched as before (with its spellings and forms).
* **Several units** must all occur in a passage (`lemma:kapota lemma:vana`); `-lemma:x` must not; `( … OR … )`
  joins alternatives; everything else of the language combines (`in:`, `-in:`, `lang:`, `type:`, `date:`,
  `author:`, `sort:date`, other words). Charts (`/api/search/chart`) count the same passages.
* **The analyses** are words.db's (`occ`: lemma → passage with its form, curated or automatic; `form.gram`, the form's
  commonest analysis; `st_form_gram`, every analysis its occurrences had). Three notations are read as one vocabulary
  (Sanskrit `LOC.M.SG` / `3.SG.PRF.IND.ACT`, Pāli DPD `masc/nt acc sg; nt nom/voc sg`, Prakrit PSM `acc.sg/nom.sg.n`).
  A form matches when **any** of its readings matches; an automatically analysed Sanskrit form also with every
  reading vidyut-kosha has for it (`jagāma`: 1st or 3rd person) — and, from the next release, without a word too
  (`form_alt`). A reading that only a few of a form's occurrences had (under 5 %) does not count for all of them:
  gata is a Vedic aorist imperative 12 times out of 2,792.
* **Results.** Each passage marks the form found (`snippet.marks` = `[start, end, analysis]`: hover it); the count line
  shows "grammar ⓘ" with what the analyses are.
* **Speed.** The filters narrow the documents first, and the lemma index is read only over their passages (their id
  runs, `docrun`). The lemma and pattern work stops after `PRAMANA_WORDQ_BUDGET` seconds (default 3.5) with the
  passages found so far: the answer says `wordq.partial` and `wordq.estimate`, the page "first N of ≈M". Patterns are
  prefiltered by the trigram index on the letters every match must contain (character classes and alternations of a few
  letters multiply out: `agn[iy]` → agni OR agny), then checked on the stored normalised text (`seg_norm`), originals
  first, machine translations last; a pattern without three letters in a row reads every passage of its scope (a work,
  up to 300,000 passages). Timings: docs/work-2026-10i/S2.md.
* **Limits.** One occurrence per lemma and passage is recorded (the first, curated before automatic): a passage where
  the lemma occurs twice is found by the form recorded. Automatic analyses are a tagger's (vidyut-kosha); voice is not
  given for automatic Sanskrit verbs. Tamil forms carry no analysis. Patterns see the normalised text, so `\b` falls
  between words as printed (sandhi joins are one word).

## Search by meaning

`pramana/semantic.py`. Each passage of a work's own text and of each commentary is embedded with the English aligned
to it (a human translation, else the machine translation) and its work's title; passages of English translations
that align with nothing are embedded on their own. A query is embedded through OpenRouter (`OPENROUTER_API_KEY`, a
fraction of a cent a search) and compared with every passage the filters allow: first by the signs of the vectors,
then exactly for the best 4,000. The answer's `meaning` block lists the nearest passages with their English, and for a
query of up to four words the dictionary words whose glosses mean it (`semantic-words.idx`, the glosses of words.db).
A sentence in English is searched by meaning only (`words_skipped`, a link searches its words).

* Index files: `data/semantic.idx` (passages) and `data/semantic-words.idx` (words), memory-mapped by every worker,
  built by the `semantic` step of `pramana release` (`python -m pramana.semantic build`, `--words` for the words);
  vectors of unchanged texts are reused, so a release embeds only new and changed passages; `--budget` stops a run
  at that many dollars and the next run resumes. Model `PRAMANA_EMBED_MODEL` (the bake-off: `scripts/semantic_bakeoff.py`;
  Gemini embedding 2 best, Qwen3 8B cheapest), `PRAMANA_EMBED_DIMS` 512.
* Which texts: documents of the text qualities in `PRAMANA_SEMANTIC_QUALITY` (default `all`: OCR'd printed books and
  page-extracted editions too). The index of 2026-10-28 holds all 12,919,585 passages of the library (7.75 GB);
  Gemini embedding 2 cost $372 in all ($166 for the first 5.0M clean passages, $207 for the rest). A release embeds at
  most `PRAMANA_SEMANTIC_BUDGET` dollars (default 5) of new or changed passages and keeps the rest for the next run.
* Ranking: modern studies (`ins.`) explain the sources in fluent English and so match English questions closely; they
  get a handicap of 0.035 (`STUDIES_HANDICAP`) so that the sources come first unless a study is clearly nearer. The
  first pass rescoring keeps 2,000 rows (`PRAMANA_SEMANTIC_RESCORE`: 97% of the top ten of 4,000, half the random
  reads of the int8 section, which the server cannot cache); the sign bits are scanned as 64-bit words in 4 threads.
* **Related passages** (`/api/search/related`, `pramana/search/expand.py`): for an English question of two or more
  words whose best passage scores below 0.84, a model (`PRAMANA_EXPAND_MODEL`, Gemini 3.8 Flash, ≈ $0.0012 a question)
  names up to six concrete passages, episodes or terms it may be about (`man conquers nature` → Pṛthu levelling the
  earth, the Khāṇḍava forest, Bhagīratha …; `original research` → Pāṇini's upajñā); each is searched by meaning and
  the passages found are shown under "related". The model's sentences are never shown (the page used to print them
  as headings, which read like quotations; they can name a work or an episode wrongly): `phrase` in the API only. At most $20 a calendar month (`PRAMANA_EXPAND_MONTHLY_USD`; the ledger is the `spend` table
  of data/user.db), answers kept (`expand_cache`): a question asked again costs nothing.
* Without an index or a key the block is absent and search is as before.
* Limits: a work without any English (human or machine) is embedded from its original alone and is found by an
  English question far less reliably; a term whose sense differs from its gloss's words (`original research` →
  upajñā "knowledge found out by one's self … invention") is not reliably found among the words either.
