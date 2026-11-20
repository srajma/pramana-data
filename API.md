# Pramāṇa API — examples

Base URL: `https://indike.org/pramana/api`. Interactive docs: https://indike.org/pramana/api/docs. JSON everywhere,
no key. Please send no more than a few requests a second; for bulk work use the open data release instead.

## Documents and passages

| request | returns |
|---|---|
| `GET /page/{path}` | everything a page shows: titles, dates, authors, recensions, commentaries, translations, text |
| `GET /tree/{path}` | the children of a category or the sections of a work |
| `GET /segments/{path}?offset=&limit=` | the passages under a path, with their locators |
| `GET /cite/{path}` | Chicago, MLA and BibTeX citations |
| `GET /person/{id}` | a person with their works |
| `GET /works` | the major works of each section |
| `GET /about/facts` | the numbers on the About page |

## Search

`GET /search?q=…` takes one query line: words in any script or spelling, `in:<path>` to restrict to a branch,
`type:commentary`, `lang:pi`, `date:[-500,0]`, `author:…`, `-word`, `( … OR … )`.

```sh
curl 'https://indike.org/pramana/api/search?q=karmaṇy+evādhikāras+in:in.darsana.all.gita'
```

## SQL

`POST /sql` with `{"sql": "…"}` runs any read-only SQLite query over the whole database (at most 10,000 rows a
query). `GET /schema` lists the tables. The main ones:

| table | holds |
|---|---|
| `doc` | works, recensions, commentaries, translations and categories: `path`, `kind`, `title`, `title_en`, `lang`, `status`, `date_s`/`date_e` (composition window, years CE, negative = BCE), counts |
| `segment` | the passages: `doc_id`, `loc` (locator), `parent`, `ord`, `text`, `nwords` |
| `relation` | `comments_on`, `translates`, `recension_of`, `parallel`, `quotes`, `contains`, `based_on` between documents and passages |
| `claim` | dates, authors and places with their scholarly `source_id` |
| `provenance` | where each text came from: repository, URL, edition, licence |
| `person`, `place`, `source` | people, places, the bibliography of claims |

Example — Sanskrit words by century of composition:

```sql
SELECT CAST(date_s / 100 AS INT) * 100 AS century, sum(nwords) AS words
FROM doc WHERE lang = 'sa' AND kind = 'text' AND date_s IS NOT NULL
GROUP BY century ORDER BY century
```

## Markdown pages

Any page as Markdown: `https://indike.org/pramana/<path>?format=md` (or `Accept: text/markdown`).
