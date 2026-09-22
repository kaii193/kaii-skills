# Domain-specific source hierarchy and distinctions

Read the section for the domain you're researching before you start pulling sources. Each one lists what to prefer and what to keep visibly separate in the writeup.

## AI / AI models

Prefer: official model documentation, model cards, official research papers, official API documentation, official benchmark results, official pricing documentation, official release notes.

Keep separate:
- Officially documented capability (the provider states it does X)
- Observed benchmark result (a specific score on a specific eval)
- Independent evaluation (a third party ran their own test)
- Inference/interpretation (you concluding something from the above)

Never present a benchmark result as universal real-world performance — a benchmark is a score on that benchmark, not a guarantee of behavior on the user's task.

## Finance

Prefer: regulatory filings, central banks, government agencies, official exchange data, issuer reports, audited financial statements, official fund documents and prospectuses, primary market data.

Keep separate: fact, historical data, issuer statement, market data, analyst interpretation, risk consideration.

Never turn the research note into unsupported investment advice — state what's documented and what's analysis, and let the reader decide.

## Science

Prefer: original research papers, official datasets, standards, institutional publications, authoritative scientific organizations.

Keep separate: established evidence, a single study's finding, a hypothesis, interpretation, and stated uncertainty. A single study finding is not established evidence until it's been replicated or is treated as such by the field.

## Technology / software

Prefer: official documentation, specifications, RFCs, source code, official repositories, release notes, official API references.

Verify version-specific behavior explicitly — "this is true as of vX" — since docs and behavior both drift across versions.

## Products / services

Prefer: official product documentation, official pricing, official terms, product specifications, first-party announcements.

Keep separate: documented features (what the vendor states it does) from user reviews or third-party opinions about how well it does it.

## Claim ownership — "who owns this information?"

For any claim not covered above, ask who owns it before citing a secondary source:

| Claim | Source of truth |
|---|---|
| Model capability | Model provider |
| Model architecture | Research paper / provider |
| API parameter or behavior | API specification |
| Product feature | Product documentation |
| Financial result | Financial filing |
| Market price | Exchange / authoritative market data |
| Regulation | Government / regulator |
| Scientific finding | Original research |
| Protocol behavior | Specification / RFC |

If you can't trace a claim to its owning source, say so in the note rather than citing whatever secondary source happened to repeat it.
