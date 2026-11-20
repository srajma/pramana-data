# Pramāṇa — an open database of Indian literature and primary sources

**[Pramāṇa](https://indike.org/pramana/)** is an open, comprehensive database of Indian literature and primary
sources: texts in their original languages (Sanskrit, Pāli, Prakrit, Tamil and others) with their recensions,
commentaries and translations linked verse by verse, inscriptions, coins, archaeological sites and museum objects,
what other civilisations wrote about India, and classic studies. It is free to read and to query, with no
registration.

This repository is Pramāṇa's public home on GitHub: what the database holds, how to use its API, how to cite it,
the open data releases, and where to report errors. (The application's code is kept in a separate repository.)

## In numbers (https://indike.org/pramana/about, build of 2026-11-14)

| | |
|---|---|
| Works catalogued | 205,407 |
| Works with their text | 51,241 |
| Passages | 18,476,537 |
| Words (original texts) | 574,784,749 |
| Recensions and editions | 12,298 |
| Commentaries | 23,199 |
| Translations (human) | 29,269 |
| Machine translations (marked as such) | 23,895 |
| Languages | 93 |
| Inscriptions | 67,417 |
| Archaeological sites | 24,952 |
| Museum artefacts | 195,053 |

## What to try

* A verse with all its commentaries and translations: the Bhagavadgītā 2.47 —
  https://indike.org/pramana/in.darsana.all.gita.2.47
* The Ṛgveda with Sāyaṇa, Skandasvāmin, Udgītha and Veṅkaṭamādhava, verse by verse —
  https://indike.org/pramana/in.veda.samhita.rg
* The Nyāya Sūtras with Vātsyāyana's Bhāṣya and the later commentaries —
  https://indike.org/pramana/in.darsana.nyaya.pracina.nyayasutra
* The major works of each section — https://indike.org/pramana/works
* Search in any script or spelling (Devanāgarī, IAST, Harvard-Kyoto, or plain "krishna") —
  https://indike.org/pramana/search?q=dharmakṣetre

## Paths

Every document has a permanent path that is also its URL. Old paths keep working (they redirect).

| path | what |
|---|---|
| `in.veda.samhita.rg` | the Ṛgveda (a work) |
| `in.veda.samhita.rg.1.1.1` | its first verse |
| `in.veda.samhita.rg!sayana` | Sāyaṇa's commentary on it |
| `in.veda.samhita.rg~en-griffith` | Griffith's translation |
| `in.veda.samhita.rg*sakala` | the Śākala recension |
| `in.…` Literature · `inp.…` Physical (sites, monuments, inscriptions, coins, artefacts) · `inf.…` Foreign · `ins.…` Studies | the four sections |

Append `?format=md` to any page URL for Markdown. `/llms.txt` and `/llms-full.txt` describe the site for
language models.

## API

Interactive documentation: https://indike.org/pramana/api/docs (OpenAPI: `/pramana/api/openapi.json`). No key is
needed; please keep to a few requests a second. See [API.md](API.md) for examples.

```sh
# the passages of Ṛgveda 1.1
curl 'https://indike.org/pramana/api/segments/in.veda.samhita.rg.1.1'
# a citation (Chicago, MLA, BibTeX) for any passage
curl 'https://indike.org/pramana/api/cite/in.veda.samhita.rg.1.1.1'
# read-only SQL over the whole database (schema: /pramana/api/schema)
curl -X POST 'https://indike.org/pramana/api/sql' -H 'content-type: application/json' \
  -d '{"sql": "SELECT lang, count(*) AS works, sum(nwords) AS words FROM doc WHERE kind = \"text\" AND nwords > 0 GROUP BY lang ORDER BY words DESC LIMIT 10"}'
```

## Open data

The openly licensed part of the database (public-domain editions, CC0, CC BY and CC BY-SA texts, and Pramāṇa's own
catalogue, dates and alignments) is being released as Parquet tables on Zenodo and Hugging Face; the links will
appear here.

Every passage carries the licence class of its text, and `catalogue.parquet` names each source and its licence
statement. Texts that their sources allow only for non-commercial use, or show with permission, are not in the
release. They can be read on the site.

## Sources

Pramāṇa reads the texts of GRETIL, SARIT, the Digital Corpus of Sanskrit, SuttaCentral, CBETA, 84000, Muktabodha,
Ambuda, the Jain eLibrary, Wikisource, the Internet Archive's scans (read by OCR) and many other projects. It credits
each on every page that shows its text, with the edition and the licence. The full list:
https://indike.org/pramana/about#sources. Machine translations and OCR-derived texts are marked as such.

## How to cite

> Pramāṇa: an open database of Indian literature and primary sources. https://indike.org/pramana/, 2026.

Cite a passage by its path, e.g. https://indike.org/pramana/in.veda.samhita.rg.1.1.1. The API gives a ready
citation for any passage (`/api/cite/<path>`).

## Errors, missing texts, suggestions

Open an [issue](../../issues) here, or use "Propose" on any passage of the site to correct it. Missing editions and
badly read texts are the most useful reports.
