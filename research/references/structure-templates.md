# Structure templates by domain

These are starting shapes, not rigid molds — add or drop sections when the subject calls for it. The default in SKILL.md covers most cases; use these when the domain has a more natural shape.

## Default (general technology, standards, protocols, business concepts)

```markdown
# <Topic>

## WHAT
## WHY
## CONCEPT
## CODE EXAMPLE            <!-- only when code helps explain it -->
## API DOCUMENTATION        <!-- only when it exposes an API/SDK/protocol -->
## LIMITATIONS / CONSIDERATIONS
## SOURCES
```

## Financial products / markets

```markdown
# <Financial Topic>

## WHAT
## WHY
## CONCEPT
## HOW IT WORKS
## RISK
## HISTORICAL DATA
## LIMITATIONS
## SOURCES
```

## AI models

```markdown
# <Model>

## WHAT
## WHY
## CONCEPT
## CAPABILITIES
## LIMITATIONS
## BENCHMARKS
## CODE EXAMPLE
## API DOCUMENTATION
## SOURCES
```

## API DOCUMENTATION section — what to include when present

| Item | Description |
|---|---|
| Endpoint / method | The API operation |
| Purpose | What it does |
| Authentication | Required auth |
| Parameters | Required / optional |
| Request | Request format |
| Response | Response format |
| Errors | Relevant error cases |
| Limits | Rate limits / constraints |
| Version | Applicable version |
| Source | Official documentation link |

Omit the whole section for subjects without an API — don't force an empty table.
