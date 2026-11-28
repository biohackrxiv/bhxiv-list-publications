# BioHackrXiv: the list of accepted publications

This repository is the **authoritative list of papers BioHackrXiv has accepted**.
Zenodo holds the papers; this holds the fact that we said yes, when, and to which
exact file.

One JSON-LD file per accepted version, in `papers/`, named after the DOI. `index.json`
is generated from them and must not be edited.

## Why this exists

A Zenodo record cannot say "BioHackrXiv accepted this" — Zenodo does not know about
us. It is also the source of truth for everything *descriptive*: authors, abstract,
licence, files, publication date. So this repository stores only what Zenodo cannot:

> Which exact PDF, of which paper, when, and that it was accepted. Everything else
> resolves from the DOI.

Copying authors or abstracts here would create a second version of the truth that
goes stale the moment a published record's metadata is edited.

## The records

`papers/10.5281_zenodo.24497647.json` (the DOI's slash is flattened to `_`):

```json
{
  "@context": "https://biohackrxiv.org/ns/ledger-context.jsonld",
  "@type": "schema:ScholarlyArticle",
  "@id": "https://doi.org/10.5281/zenodo.24497647",

  "identifier": "10.5281/zenodo.24497647",
  "title": "BioHackEU23 report: …",
  "type": "Text",
  "publisher": "BioHackrXiv",
  "event": "accepted",
  "dateAccepted": "2026-11-28T09:00:24+00:00",
  "pdfSha256": "c9169ada1a5e53aeb839dda6907977269b35442a2b1500c4c9f205dcc635106c",
  "isVersionOf": "https://doi.org/10.5281/zenodo.24466632"
}
```

| field | predicate | meaning |
|---|---|---|
| `@id` | — | the DOI URL: the record *is* the resource, not a description of one |
| `identifier` | `dcterms:identifier` | the DOI; agrees with the file name |
| `title` | `dcterms:title` | so the list is readable and greppable without a network |
| `type` | `dcterms:type` | `dcmitype:Text` |
| `publisher` | `dcterms:publisher` | the venue that accepted it |
| `event` | `bhxiv:ledgerEvent` | `bhxiv:Accepted` or `bhxiv:Withdrawn` |
| `dateAccepted` | `dcterms:dateAccepted` | the editorial decision |
| `dateWithdrawn` | `bhxiv:dateWithdrawn` | only when `event` is `withdrawn` |
| `description` | `dcterms:description` | the note explaining a withdrawal |
| `pdfSha256` | `bhxiv:pdfSha256` | binds the acceptance to the exact deposited bytes |
| `isVersionOf` | `dcterms:isVersionOf` | the concept DOI: which paper this is a version of |

The vocabulary is Dublin Core Terms, with schema.org only for the type and four
terms of our own in `ledger-context.jsonld` — which is also copied here, because a
ledger whose only vocabulary lived on a website would be unreadable the day that
website changed.

## Reading it

```sh
jq -r '"\(.dateAccepted)  \(.identifier)  \(.title)"' papers/*.json | sort
```

or walk the generated aggregate:

```sh
curl -s https://raw.githubusercontent.com/biohackrxiv/bhxiv-list-publications/main/index.json | jq '.hasPart[] | .identifier'
```

## The rules

- **One file per accepted version.** A new version needs its own acceptance,
  otherwise "accepted" would transfer to a paper nobody reviewed.
- **Never edit a record by hand.** They are written by `xiv-publish`, which derives
  every field from the submissions database.
- **A withdrawal rewrites that paper's file** (`event: withdrawn`, keeping
  `dateAccepted`); the diff is the record of the change. Withdrawn papers are left
  out of `index.json`.
- **`index.json` is generated.** CI regenerates it; do not edit it.
- **This is append-only in spirit**: files are added, and the only thing that
  changes an existing file is a withdrawal.

## Checking

CI runs, on every push:

```sh
xiv-publish ledger verify .
xiv-publish ledger index .
```

`verify` checks that every key in every record is defined in the context (JSON-LD
fails quietly, so a misspelled key would otherwise be a property nobody
understands), that the required fields are present, that `@id`, `identifier` and
the file name agree, that dates carry a timezone, and that `pdfSha256` is a real
digest.

## Provenance

The tool that writes this, and the full design note, are in
[biohackrxiv/xiv-publish](https://github.com/biohackrxiv/xiv-publish)
(`doc/ledger.md`). The decision itself is recorded in the submissions database of
the preview service; this repository is its public consequence.
