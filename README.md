# Your team's knowledge graph system

The starter repository for the Knowledge Representation team project
(Lebanese University, MSc). Every session of the course adds one layer; this
repository is where your team puts its own version of each, on its own
topic and data. In Session 8 the instructor clones it fresh and runs one
command. What comes up is what is graded.

It ships with a tiny real example (60 Brunel orders from the course's shared
case), so the whole stack comes up on the first day. Replace the example,
one layer at a time, with your own.

## Run it

```sh
cp .env.example .env        # then put your Gemini key after GOOGLE_API_KEY=
docker compose up --build
```

**Expect:** the pipeline prints five steps and `done` (on the example:
1,185 triples, 0 Violation, 189 VoID triples), then the access layer serves
on http://127.0.0.1:8000. Try
`http://127.0.0.1:8000/?question=How%20many%20late%20orders%20are%20there%3F&dataset=https://ul.edu.lb/kr/brunel/`.
The endpoint is at http://127.0.0.1:3030 (dataset `kr`).

Without Docker, the pipeline also runs on its own (Python 3.12, from the
repository root): `pip install -r pipeline/requirements-pipeline.txt`, then
`pip install --no-deps morph-kgc==2.10.0`, then `python pipeline/run.py --no-load`.

## What goes where

| Folder | Layer | Session | Replace the example with |
|---|---|---|---|
| `data/` | your source, or a small sample of it | 1, 5 | a sample `example.sql`, or set `DB_URL` in `.env` to your database |
| `ontology/` | the ontology | 3 | your ontology: a reused published vocabulary, competency questions, OWL profile, version IRI |
| `shapes/` | the SHACL shapes | 4 | the shapes for your constraint inventory |
| `mapping/` | the R2RML mapping | 5 | the mapping from your tables to your ontology |
| `pipeline/` | map, gate, describe, load | 4, 5 | usually nothing |
| `model/` | the graph models | 6 | your node classification and link prediction, with baseline and split |
| `access-layer/` | questions in English | 7 | `prompt_rules.txt`, `examples.ttl`, `questions.yaml`, and `NS`/`PFX` in `access_layer.py` |
| `report/` | the technical report | all | your report, from `REPORT.md` |
| `.github/workflows/` | the SHACL gate in CI | 4 | nothing, if `data/example.sql` holds a sample of your data |

## How the stack fits together

```text
data (your tables) --> pipeline: map (mapping/) --> SHACL gate (shapes/ + ontology/)
                                                        |
                                    Violation: stop     |  pass
                                                        v
                                   VoID (build/void.ttl) + load --> Fuseki (kr)
                                                                        ^
                   a question in English --> access layer (Gemini) -----+
```

The endpoint is only ever loaded with data that passes the gate. Change one
row of `data/example.sql` so an order weighs 0 and run the pipeline again: it
stops at step 3 and says why.

## The Session 8 checklist

The checklist agreed in Session 7; the defense tries each item on a clean
checkout:

- one command starts the whole stack from a clean checkout;
- every service listens on 127.0.0.1 only, and keys live only in `.env`;
- the SHACL gate runs before data reaches the endpoint (and in CI);
- the test set's score is recorded, with the model and the date;
- a real unanswerable question is refused, with a reason.

## Rules the course holds you to

- Commit under your own account; one branch and one pull request per person,
  reviewed by a teammate.
- Never commit `.env`, a key or a password.
- Every third-party ontology, vocabulary and dataset goes in the report with
  its licence.
- Any teammate can be asked about any part, in Session 8.
