# BioHackrXiv: the list of accepted publications

This repository is the **authoritative list of papers BioHackrXiv has accepted**.
Zenodo holds the papers; this holds the fact that we said yes, when, and to which
exact file.

One JSON-LD file per accepted version, in `papers/`, named after the DOI.
`index.json` is generated from them and must not be edited.

## Why this exists

A Zenodo record cannot say "BioHackrXiv accepted this" — Zenodo does not know about
us. It is also the source of truth for everything *descriptive*: authors, abstract,
licence, files, publication date. So this repository records only what Zenodo cannot:

> Which exact PDF, of which paper, when, and that it was accepted. Everything else
> resolves from the DOI.

Copying authors or abstracts here would create a second version of the truth that
goes stale the moment a published record's metadata is edited.

## The records

`papers/10.5281_zenodo.24497647.jsonld` (the DOI's slash is flattened to `_`):

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

The extension is `.jsonld` because that is what the contents are.  It is JSON — `jq`
reads it — but the keys are not private names: each one resolves, through
`ledger-context.jsonld`, to a term somebody else defined.

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

The vocabulary is Dublin Core Terms, with schema.org only for the type and four terms
of our own, declared in `ledger-context.jsonld` — which is also copied here, because a
ledger whose only vocabulary lived on a website would be unreadable the day that
website changed.

Expanded, the record above becomes what a triples store or an OAI harvester can merge
with whatever else it knows about that DOI:

```
<https://doi.org/10.5281/zenodo.24497647>
    rdf:type                              schema:ScholarlyArticle
    http://purl.org/dc/terms/identifier   10.5281/zenodo.24497647
    http://purl.org/dc/terms/title        BioHackEU23 report: …
    http://purl.org/dc/terms/publisher    BioHackrXiv
    http://purl.org/dc/terms/dateAccepted 2026-11-28T09:00:24+00:00
    https://biohackrxiv.org/ns#ledgerEvent https://biohackrxiv.org/ns#Accepted
    https://biohackrxiv.org/ns#pdfSha256  c9169ada…
    http://purl.org/dc/terms/isVersionOf  https://doi.org/10.5281/zenodo.24466632
```

## Reading it

```sh
jq -r '"\(.dateAccepted)  \(.identifier)  \(.title)"' papers/*.jsonld | sort
```

or walk the generated aggregate:

```sh
curl -s https://raw.githubusercontent.com/biohackrxiv/bhxiv-list-publications/main/index.json \
  | jq '.hasPart[] | .identifier'
```

`index.json` is a `schema:Dataset`: per accepted version only `@id`, `identifier`,
`title`, `dateAccepted` and `isVersionOf`.  Withdrawn papers are left out.  A harvester
that wants the rest resolves the DOI.

## The rules

- **One file per accepted version.** A new version needs its own acceptance, otherwise
  "accepted" would transfer to a paper nobody reviewed.
- **Never edit a record by hand.** They are written by `xiv-publish`, which derives
  every field from the submissions database.
- **A withdrawal rewrites that paper's file** (`event: withdrawn`, keeping
  `dateAccepted` and adding `dateWithdrawn` and the reason); the diff is the record of
  the change. That is what happens when a decision is reversed within seven days, or
  when the Zenodo record is deleted.
- **`index.json` is generated.** CI regenerates it; do not edit it.
- **This is append-only in spirit**: files are added, and the only thing that changes
  an existing file is a withdrawal.

## Checking

CI runs, on every push:

```sh
xiv-publish ledger verify .
xiv-publish ledger index .
```

`verify` checks that every key in every record is defined in the context (JSON-LD
fails quietly, so a misspelled key would otherwise be a property nobody understands),
that the required fields are present, that `@id`, `identifier` and the file name agree,
that dates carry a timezone, that `pdfSha256` is a real digest — and that nothing in
`papers/` is a file it does not recognise, since a silently ignored file is how a
renamed extension turns into an empty index.  The workflow then fails if the
regenerated index does not account for every record.

## Provenance

The tool that writes this, and the full design note, are in
[biohackrxiv/xiv-publish](https://github.com/biohackrxiv/xiv-publish)
(`doc/ledger.md`). The decision itself is recorded in the submissions database of the
preview service; this repository is its public consequence.
