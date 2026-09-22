---
name: research
description: Investigate a topic or question from primary and authoritative sources — not secondhand summaries — and produce one evidence-backed Markdown research note. Domain-agnostic: use for AI/LLM and model research, finance and financial products/markets, technology and software/APIs/protocols, products and services, business concepts, scientific concepts, and standards/regulations/policy. Trigger this whenever the user asks to research, investigate, look into, dig into, or find out about a topic, wants a comparison or writeup of options, or asks a substantive "what is X" / "how does Y work" / "is Z true" question that needs sourcing — even if they never say the word "research". Not for quick lookups you can answer directly from something already open, and not for pure code-writing tasks (see gf-start-coding for those).
---

# Research

Understand a subject from its **source of truth**, not from summaries of it. The output is one Markdown file a reader can trust because every important claim traces back to whoever actually owns that information.

## Job, in order

1. Restate what's actually being asked — a research question can hide two or three narrower ones. If genuinely ambiguous in a way that would change the answer, name the interpretations and pick the one that matters most, or ask.
2. For each important claim you'll need, identify who owns that information (see `references/source-hierarchy.md` for the source-of-truth table by domain).
3. Investigate primary sources first. Fall back to authoritative secondary sources only where primary is silent or unavailable. Use general secondary sources (blog posts, forum threads, generic articles) only to find terminology or leads to a primary source — never as the final citation for a claim primary sources could answer.
4. Trace each important claim back to that owning source as you write, not as an afterthought — cite inline, at the point the claim is made, not only in a bibliography at the end.
5. Keep facts, source claims, analysis, inference, and recommendation visibly separate (see Facts vs Analysis below). A reader should always be able to tell "this is documented" from "this is my reasoning about what's documented."
6. Check version and time sensitivity for anything that changes: model versions, pricing, API behavior, regulations, financial data, product specs. State what version/date the claim applies to. Never present something historical as current without checking.
7. Write the findings to **one** Markdown file, structured for the domain (see Structure below).
8. Report the exact file path in your final response.

## Source hierarchy

1. **Primary** — official docs, specs, standards, regulatory filings, government publications, official datasets/APIs/source code, official research papers, first-party announcements and financial reports, model cards, official benchmarks, official terms/policies. Use these whenever they exist.
2. **Authoritative secondary** — academic publications, reputable financial institutions, established research organizations, recognized industry bodies, high-quality technical publications. Use when primary sources don't cover the claim.
3. **General secondary** — everything else. Use only to discover terminology or find a primary source, never as the final authority for a claim a primary source could settle.

Domain-specific preferred sources, and what to distinguish within each domain (AI/models, finance, science, tech/software, products) are in `references/source-hierarchy.md` — read it before researching any of those domains.

## Core questions

Answer whichever of these are relevant to the subject — don't force a section that doesn't fit.

- **WHAT** — what it is, what it does, its scope, main components, capabilities, limitations.
- **WHY** — why it exists, what problem it solves, why it's used, documented trade-offs, appropriate and inappropriate use cases. Keep documented rationale ("according to the official docs, X was introduced to...") visibly separate from your own analysis ("from that behavior, this implies...").
- **CONCEPT** — how it's structured or works, adapted to the domain (architecture/components for tech; training/inference/context for AI models; instrument/pricing/risk for finance; actors/process/business model for business; scope/definitions/enforcement for regulation).

Include a **CODE EXAMPLE** section only when code materially helps explain the subject (APIs, SDKs, algorithms, protocols, libraries) — never invent an API, parameter, or response; cite the source for non-obvious behavior. Include an **API DOCUMENTATION** section only when the subject exposes an API/SDK/protocol. Omit both for financial, business, regulatory, historical, or general product topics where they add nothing.

## Facts vs analysis

Never let analysis or inference read as directly documented fact. Distinguish:

- **Fact** — directly supported by a source.
- **Source claim** — an assertion made by an organization, company, researcher, or other party (attribute it to them, not to reality).
- **Analysis** — your reasoning based on available evidence.
- **Inference** — a conclusion drawn from combining multiple facts.
- **Recommendation** — an engineering, financial, or practical suggestion, clearly labeled as such.

## Citation

Cite at the point the claim is made, not only in an end bibliography:

> The model supports a 1M-token context window according to the provider's documentation. [Source](...)

A reader should be able to answer "where did this statement come from?" for every important factual claim without hunting.

## Research depth

- **Simple question** — enough authoritative sources to establish the answer; don't pad.
- **Complex question** — multiple primary sources, different versions/specs, contradictory documentation, important limitations, historical changes.
- **Ambiguous question** — name the ambiguity and investigate the interpretations that would actually change the answer.

## Output location

Before writing, check the working directory for an existing docs/research convention (a `docs/research/`, `research/`, or similar folder; existing file naming pattern) and follow it. If none exists, use `docs/research/<topic>.md` (kebab-case topic) as a sensible default and say so in your response — don't invent a convention no one asked for.

## Structure

Default shape — adapt section names to the domain rather than forcing these onto a subject they don't fit. Full worked variants (financial, AI-model) are in `references/structure-templates.md`.

```markdown
# <Topic>

## WHAT
## WHY
## CONCEPT
## CODE EXAMPLE            <!-- only when applicable -->
## API DOCUMENTATION        <!-- only when applicable -->
## LIMITATIONS / CONSIDERATIONS
## SOURCES
```

## Before finishing, check

- Every important claim is cited at the point it's made, and traced to whoever owns that information.
- Facts, source claims, analysis, inference, and recommendation are visibly distinct.
- Version/date sensitivity is checked wherever the answer could have changed.
- CODE EXAMPLE / API DOCUMENTATION sections appear only where they earn their place.
- The file follows the repo's existing docs convention, or you said why not.
- Your final response states the exact file path.
