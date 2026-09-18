# Maisa Hackathon · 500 Sombras de Alberto

Team **`'; DROP TABLE Hackers; /*`** · MAISA track · HackSpain 2026 · 18–20 Sep 2026 · ETSIT UPM, Madrid

> Alberto processes invoices, a chaotic Excel and a 2009 ERP. Today it is 500 invoices; tomorrow he
> wants new file types, more volume, and answers even when a model provider is down. The job is not
> to win a benchmark: it is to build a system Alberto can trust with a real operation.

- Challenge site: https://hackathon.maisa.ai/
- Starter data ("La Caja"): https://github.com/ikurotime/500-sombras-de-alberto
- Event CLI: https://hackspain.app/cli (see `.claude/skills/hackspain-cli/SKILL.md`)

## The task

For every invoice PDF, decide **`PAGAR`**, **`NO_PAGAR`** or **`ESCALAR`** by reconciling it against:

| Source | What it is |
| --- | --- |
| `facturas/` | 500 invoice PDFs — many layouts, two number formats, 29 image-only scans/faxes |
| `FINAL_v7_DEFINITIVO_ahorasi.xlsx` | Supplier master (`Proveedores`), orders (`Pedidos_2026`), the payment rule (`Norma_Pagos_v3`) and ~10 junk sheets |
| `alberto_erp.py` | Local ERP bridge on `127.0.0.1:8009` — XML, ISO-8859-1, paginated, flaky by design. The official accounting reference |
| `MANUAL_ERP_2009.md` | ERP manual: login, endpoints, error codes |

The format is free (CLI, backend, agent tool, web app…). **Choose it because it fits the problem and
be ready to defend why.** All data is synthetic.

### Payment rule v3 (from the Excel, verbatim)

1. Pagar solo si el NIF esta en el maestro y el IBAN de la factura coincide con el maestro.
2. El pedido debe existir, pertenecer al proveedor y el importe de la factura debe ser igual al del pedido (tolerancia 0,01 EUR).
3. El IVA debe estar bien calculado y el total debe ser base + IVA, con la misma tolerancia de 0,01 EUR.
4. La fecha debe ser valida y no futura.
5. Estado ERP del pedido: PENDIENTE. Nunca pagar dos veces el mismo pedido.
6. Cualquier anomalia que un humano deba ver: ESCALAR con motivo. Ante duda razonable, escalar antes que pagar.

**Rule v4 arrives Saturday 18:00** — rules must be versioned config, not hard-coded.

## Timeline (Madrid time)

| When | What |
| --- | --- |
| Fri 18 · 21:00 | Statement, rubric, `caja-de-alberto-v3.2.zip` released in the track channel (verify hashes) |
| Sat 19 · 18:00 | Batch 2: 40 more invoices, ERP update (`erp_export_lote2.csv`), rule v4, surprise scenario |
| **Sun 20 · 10:30** | **Delivery.** The starter README says 10:30, the website says 11:00 — **treat 10:30 as the deadline** |
| Sun 20 | Alberto may change one datum in La Caja to check the demo is real → reprocessing must be cheap |
| Sun 20 | Defenses, 10 min per team, single jury |

## Deliverable

A **separate public GitHub repo** (not this one — no solution code, credentials or runnable app in
it) whose root contains exactly:

```text
la-caja-outcomes/
├── outcomes.jsonl          # batch 1 — one line per file
├── outcomes_lote2.jsonl    # batch 2
└── albertitos_plan.pdf     # Arquitectura + ADRs / trade-offs
```

```json
{"file_id":"factura_5518.pdf","result":"PAGAR"}
{"file_id":"FA-4475_informática.pdf","result":"ESCALAR"}
```

Only `file_id` and `result` are mandatory; extra trace fields are allowed. Share the `teamId` and
the repo URL with the organisers. They clone it, record the commit and run a private verifier —
they never run our code.

`albertitos_plan.pdf` needs two sections:
- **Arquitectura** — components, data and state flow, split between agents / models / people, how
  failures are observed and recovered.
- **ADRs / trade-offs** — 2 to 5 decisions, each with context, alternatives, decision, accepted
  consequences and evidence.

Separately, the HackSpain event itself wants a project submission via `hackspain submit`
(name, description, repo, demo, video).

## Validation vs. scoring

**Validation is binary and gives no points**: exactly one outcome per file, and each `result` must
match the private reference. Fail it and we can still defend, but not win. No ranking is published.

Scoring happens only in the defense:

| Criterion | What it measures | Points |
| --- | --- | ---: |
| Producto, arquitectura y ADRs | Problem, format, decisions, alternatives, trade-offs | 35 |
| Escalabilidad y coste | Capacity, limits, cost formula, evolution to new inputs | 25 |
| Trazabilidad y observabilidad | Auditable decisions, state, operational signals | 20 |
| Resiliencia y recuperación | Provider failure, state, degradation, recovery | 10 |
| Calidad de ejecución | Clarity, proportion, pleasant to operate | 10 |
| Bonus: mejora adicional para Alberto | Real need, originality, implemented and shown | +10 |

Tie-breakers: scalability, then resilience, then bonus.

### Defense script (10 min)

1. **Demo y contexto (2 min)** — the solution and the concrete problem it solves (show the bonus here).
2. **Arquitectura y ADRs (2 min)** — format, architecture, agents/models/people split.
3. **Trazabilidad, observabilidad, escala y coste (4 min)** — follow one real decision end to end;
   show latency, errors, retries, pending work; files/second on which hardware; the cost formula.
4. **Resiliencia y preguntas (2 min)** — what happens when the LLM provider fails, rate-limits or
   returns garbage; how work is preserved, deduplicated, degraded and recovered.

### What they explicitly do *not* want

A prescribed stack or UI · an OCR benchmark or single-model demo · abstract scale promises without
conditions and evidence · perfect HA — they want an honest strategy for provider failure.
"A small, well-reasoned backend can beat a large application without judgement."

## What the questions imply for the design

- **Traceability**: every outcome carries extracted fields, per-rule check results, data/rule
  versions, model + prompt version, latency, cost, retries.
- **Evolution**: new file types (scans, emails, spreadsheets) should be a connector/config change,
  not a rewrite. Rules v3 → v4 should be a config change.
- **Resilience**: persistent job state, idempotent processing (no duplicates), provider fallback,
  degrade to `ESCALAR` rather than guess.
- **Reprocessing**: Saturday's batch and Sunday's data change both need cheap, targeted re-runs.
- **Untrusted input**: invoices contain prompt-injection text. See `docs/data-recon.md`.

## Getting started

La Caja is vendored in `caja/` — an unmodified export of
[ikurotime/500-sombras-de-alberto](https://github.com/ikurotime/500-sombras-de-alberto) at commit
`18d43b3` (2026-09-18). **Treat it as read-only input**: never edit files in it, so every decision
traces back to a known data version. If upstream or the channel zip changes, re-export and note the
new commit/hash here.

```bash
make -C caja erp          # ERP on http://127.0.0.1:8009  (erp-fast = no artificial latency)
make -C caja erp-status
make -C caja erp-login    # usuario=alberto clave=FACTURAS2009
```

Saturday: unpack batch 2 into `lote_2_sorpresa/` at the repo root (where `caja/Makefile` expects
it) and run `make -C caja erp-lote2`.

## Repo layout

```text
README.md                         this brief
CLAUDE.md                         rules for Claude Code in this repo
docs/data-recon.md                first-pass findings on La Caja
caja/                             La Caja, vendored and read-only (500 PDFs, Excel, ERP, manual)
.claude/skills/hackspain-cli/     skill: how to use the hackspain CLI safely
```

## Prize

A trip to Maisa's HQ in Valencia plus a keyboard for each member of the winning team.
