---
status: 'accepted'
date: 2026-09-18
decision-makers: 'David Mazarro'
---

# Review invoice decisions in a web UI

## Context and Problem Statement

Alberto must reach exactly one decision (PAGAR, NO_PAGAR, ESCALAR) for each of ~500 invoices, plus a second lot delivered later. He is non-technical and works from a folder of PDFs on his own machine.

Deciding each invoice by hand does not fit the time he has. Deciding them all automatically with no review leaves him unable to trust or correct the outcome, and ESCALAR exists precisely because some invoices need his judgement.

## Decision

Build a web UI where:

1. Alberto picks a folder of invoices from his computer.
2. The system decides every invoice automatically and produces a rationale for each.
3. A summary list shows every invoice with its decision and rationale.
4. ESCALAR rows carry a prominent call to action and stay visible until he acts on them.
5. Any invoice can be previewed inline from the list.
6. Any decision can be overridden from the list.

The system always produces a complete set of decisions. Alberto's job is review and exception handling, not data entry.

## Consequences

### What we gain

- A non-technical user can clear a full lot in one sitting.
- Rationales let Alberto accept most decisions at a glance instead of reopening invoices.
- Escalations cannot be silently skipped.
- Overrides give him final authority without blocking the automated path.

### Accepted tradeoffs

- The UI is only as trustworthy as its rationales; weak ones push Alberto back to manual checking.
- A fast review pass can rubber-stamp a wrong decision.

## Implementation Plan

Product scope only. Technical choices belong in separate ADRs.

Four surfaces must exist:

- **Folder selection**: pick a local folder, with no configuration or setup steps.
- **Processing**: visible progress while invoices are decided, so a large lot does not look frozen.
- **Summary list**: one row per invoice with file name, decision, and rationale. Escalations are prominent and distinguishable from completed work.
- **Preview and override**: open any invoice without leaving the list, and change its decision in one interaction.

Rules:

- Every invoice appears in the list with a decision and a rationale. No blanks.
- An overridden decision replaces the automated one and is marked as Alberto's.
- No step requires a manual, training, or technical help.

### Verification

- [ ] A first-time, non-technical user goes from folder selection to a fully reviewed lot unassisted.
- [ ] Every row in the summary list shows both a decision and a rationale.
- [ ] Escalations are visually distinct, and a lot with unactioned escalations is identifiable at a glance.
- [ ] Any decision can be overridden in one interaction from the summary list.
- [ ] Any invoice can be previewed without navigating away from the summary list.

## Alternatives Considered

- **Fully automated, no review**: gives Alberto no way to build trust or catch errors, and no home for ESCALAR.
- **CLI or script**: Alberto is non-technical, so a command line is a wall rather than a workflow.
- **Spreadsheet of results**: no invoice preview, so verifying a decision means hunting for the PDF by hand.
- **Write decisions into the 2009 ERP**: too slow and risky to build against a legacy system on this timeline.

## More Information

Out of scope and covered by separate ADRs: output formats, classification method, ERP integration, invoice editing, and multi-user access.

Context: Maisa hackathon "500 Sombras de Alberto" (https://hackathon.maisa.ai/).
