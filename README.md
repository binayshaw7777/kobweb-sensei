# Kobweb Sensei

Curated Kobweb knowledge — reference manual + Discord FAQ — extracted from the `#need_help` channel and organized for offline use.

## Files

| File | Lines | Content |
|------|-------|---------|
| `SKILLS.md` | 742 | Kobweb reference manual |
| `FAQ.md` | 4496 | 351 problem→solution entries from Discord |
| `RULES.md` | 61 | Compact gotchas for Routing, Styling, Silk, Server, Build, Deployment |

## Source

Raw Discord archive: `need_help-page-1.json` (13,001 messages, 195 authors).  
Processor: Python script threads conversations by time/mentions, filters for bitspittle answers with Kobweb/Kotlin code, deduplicates, outputs Fix entries.

## Local Use

```bash
# Search for a fix
grep -A 10 "### scroll" FAQ.md

# List all fix titles
grep "^### " FAQ.md

# Search by category
awk '/^### Styling/,/^### Routing/' FAQ.md | head -40
```

## Key: Entry Format

```
### Fix: Descriptive title

**Problem:** What the user asked.

**Solution:** bitspittle's answer.

```kotlin
// code if applicable

**Note:** Gotcha/constraint (if any).

--- (separator)
```

## Coverage

| Category | Entries |
|----------|---------|
| Styling | 118 |
| Routing | 55 |
| Server | 43 |
| Deployment | 28 |
| Build | 40 |
| Silk | 4 |
| Markdown | 2 |
| General | 61 |
| **Total** | **351** |

## See Also

- [Kobweb Docs](https://kobweb.varabyte.com/docs)
- [Kobweb GitHub](https://github.com/varabyte/kobweb)
- [Discord](https://discord.gg/5NZ2GKV5Cs)
