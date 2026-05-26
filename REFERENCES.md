# References

## Official Documentation

**Claude Code**

- Overview: https://code.claude.com/docs/en/overview
- Skills: https://code.claude.com/docs/en/skills
- Sub-Agents: https://code.claude.com/docs/en/sub-agents
- Hooks: https://code.claude.com/docs/en/hooks

**Claude API & Products**

- Anthropic Docs: https://docs.anthropic.com
- Claude Models: https://docs.anthropic.com/en/docs/about-claude/models/overview
- API Reference: https://docs.anthropic.com/en/api/

---

## Foundational Reading

### Boris Cherny's Claude Code Setup

**The primary source on workflow optimization**

- 📌 Full guide: https://howborisusesclaudecode.com
- 🧵 X threads:
  - Thread 1 (Jan 2026): Starting workflows, parallel sessions
  - Thread 2 (Feb 2026): Skills, CLAUDE.md, compounding memory
  - Thread 3 (Mar 2026): Hooks, MCP integration, verification patterns

Key sections to read:

1. **Plan mode** - Always start here
2. **Parallel sessions** - Run 5 terminals + 5 web instances
3. **CLAUDE.md** - Living documentation that compounds over time
4. **Hooks** - Deterministic checks (PostToolUse, PreToolUse)
5. **Sub-agents** - Isolation and restricted tool permissions
6. **Verification** - Close the feedback loop (2-3x quality improvement)

---

### Andrej Karpathy Skills

**Original concept of reusable instruction sets**

- 📦 Repository: https://github.com/multica-ai/andrej-karpathy-skills
- Concept: Named, auto-invoked prompt templates
- Inspiration for Claude Code's formal skill system

---

## Architecture & Documentation Tools

**Carta - Curated Architectural Reference**

- Purpose: Maintain architecture documentation that scales with your team
- Link: https://carta.12vectors.com/
- Use with Claude Code: Keep architecture docs in sync as agents evolve the codebase
- Complements: CLAUDE.md, team wikis, ADRs (Architecture Decision Records)

---

## Related Tools & Ecosystems

**Git & GitHub**

- Git docs: https://git-scm.com/doc
- GitHub CLI: https://cli.github.com/
- GitHub API: https://docs.github.com/en/rest

**Cloud Platforms**

- AWS S3: https://docs.aws.amazon.com/s3/
- Google Cloud Storage: https://cloud.google.com/storage/docs
- Azure Blob Storage: https://learn.microsoft.com/en-us/azure/storage/blobs/

**Data Tools**

- BigQuery: https://cloud.google.com/bigquery/docs
- jq (JSON processor): https://stedolan.github.io/jq/
- SQL basics: https://www.postgresql.org/docs/

**System Administration**

- Unix/Linux: https://pubs.opengroup.org/onlinepubs/9699919799/
- macOS: https://developer.apple.com/library/archive/documentation/OpenSource/Conceptual/ShellScripting/
- NFS: https://tools.ietf.org/html/rfc7530
- SMB/CIFS: https://en.wikipedia.org/wiki/Server_Message_Block

---

## How to Use These References

- **Learning Claude Code:** Start with official docs, then read Boris's guide
- **Deepening workflows:** Browse howborisusesclaudecode.com for specific patterns
- **Implementing examples:** Follow the examples/ README and VALIDATION.md
- **Customizing:** Adapt examples using links above for your stack (different cloud, different language, etc.)

---

## Contributing

Found a broken link or outdated reference? Open an issue or PR.

Want to add a reference? Follow this format:

```markdown
**[Title]**
- Description
- Link: https://...
```
