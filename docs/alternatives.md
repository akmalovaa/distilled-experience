# Alternatives

Similar skills and tools on GitHub, checked 2026-11-12, and the memory built into
coding agents, checked 2026-11-13; star counts are from 2026-11-12. The downsides
are judged against this skill's goal: a small set of **verified engineering lessons
and decisions**, shared across projects and agents, with confirmed facts kept apart
from guesses. For another goal (remembering every session, user preferences in a
chat app) the same trait can be an advantage.

## Built into coding agents

| Feature | Approach | Downsides for this goal |
|---|---|---|
| [Claude Code auto memory](https://code.claude.com/docs/en/memory) | On by default. Claude writes Markdown notes to `~/.claude/projects/<project>/memory/` in four types: user, feedback, project and reference. The first 200 lines or 25 KB of the `MEMORY.md` index load into every session; topic files are read on demand | One store per repository on one machine, read only by Claude Code: a lesson found in another repository or by Codex does not reach it. By design it skips what the code can show, debugging fixes included, and records no evidence or status. No search: the agent knows only what the index lists. In exchange it needs no server, and the notes are plain files you can edit |
| [Codex memories](https://learn.chatgpt.com/docs/customization/memories) | Off by default (`memories = true` under `[features]`). A background job turns idle past chats into local memory files under `~/.codex`, redacts secrets, consolidates them and injects them into later sessions; `/memories` sets per chat whether it reads or feeds memory | Generated from the whole chat, so a guess is stored the same way as a confirmed cause. The docs call the files generated state and advise against editing them by hand, which leaves no supported way to fix one wrong memory short of resetting the store. Read only by Codex on one machine |

Both suit what this skill leaves out: preferences and corrections for working with one
user in one repository. They can run alongside it.

## Hindsight-based

| Project | Approach | Downsides for this goal |
|---|---|---|
| [`hindsight-coding-agents`](https://github.com/vectorize-io/hindsight/tree/main/hindsight-integrations/coding-agents) (official, in the 60k★ Hindsight repo) | npx installer wires hooks into Claude Code, Codex, Cursor and others. One bank per repository, seeded from commits and a codebase survey; the whole session transcript is retained at the end; a `reflect` answer is injected on the first prompt. Replaces the older Claude Code plugin | Stores everything, including dead ends and unverified guesses, with no separation between them. A correction is a new `Correction: <topic>` document that relies on newer facts ranking higher, so the stale claim stays in the bank. Per-repo banks keep a lesson from reaching other projects unless you map paths by hand. Defaults to Hindsight Cloud (`serverMode: cloud`) and updates itself from npm daily (`autoUpdate`) |
| [`hindsight-local`](https://github.com/vectorize-io/hindsight/blob/main/skills/hindsight-local/SKILL.md) (official skill) | `uvx hindsight-embed` CLI; tells the agent to store "always" and to pass full context, transcripts included | Its workflow has no entry format or stable document id: a revised lesson becomes one more memory next to the old one. Encourages volume over verification |
| [`agent-plugin/hindsight-memory`](https://github.com/vectorize-io/hindsight/blob/main/hindsight-integrations/agent-plugin/skills/hindsight-memory/SKILL.md) (official skill) | One page on when to call MCP `recall`, `retain` and `reflect` | Guidance only. It never sets `document_id`, so a corrected lesson is stored next to the old one instead of replacing it |
| [`hindsight-skills`](https://github.com/vectorize-io/hindsight-skills) (official, 21★) | `hindsight-architect`, `hindsight-docs`, `hindsight-upgrade`: help a developer add Hindsight to an application | Not about the agent's own memory. `hindsight-docs` is a useful API reference. Last pushed 2026-04-17 |
| Personal skills: [rjcnd105/hj-dotfiles](https://github.com/rjcnd105/hj-dotfiles), [JackSmack1971/CCSIF](https://github.com/JackSmack1971/CCSIF), [jar-reis/hermes-skills](https://github.com/jar-reis/hermes-skills) | `/hindsight save` of a whole session; trace ingestion that requires a source file and line on every record; Hindsight as one of four memory layers | Tied to their authors' setups; none has an entry schema, evidence or decision statuses. The source-and-line requirement in CCSIF is a good provenance gate |

## Curated lessons and decisions without Hindsight

| Project | Approach | Downsides for this goal |
|---|---|---|
| [`ce-compound`](https://github.com/EveryInc/compound-engineering-plugin/blob/main/skills/ce-compound/SKILL.md) in Compound Engineering (26k★) | The closest in spirit: document only a solved and verified problem, prefer the cheapest durable form (a test, lint rule or standard before a learning), update instead of duplicating, and a `refresh` mode that audits stale learnings. Learnings are `docs/solutions/*.md` with a YAML schema; a researcher agent greps them before planning | Lives in one repository, so a lesson does not reach other projects. Search is keyword-based (grep over frontmatter), with no semantic recall or cross-entry consolidation. Comes with the rest of a large plugin; `refresh` can delete docs and open a PR. In exchange, entries are reviewed in PRs and versioned with the code |
| [`architecture-decision-records`](https://skills.sh/affaan-m/everything-claude-code/architecture-decision-records) in everything-claude-code, and similar ADR skills | Decisions as ADR files: proposed → accepted → deprecated or superseded, with a link to the replacement | Decisions only: no gotchas or verification recipes. Per repository, no search beyond files |
| [ByteRover CLI](https://github.com/campfirein/byterover-cli) (4.9k★) | Curated "context tree" with a review workflow (approve or reject what the agent wants to store), git-like versioning, cloud sync | Its own CLI, REPL and LLM-provider stack to adopt; built around team and cloud sync. Last pushed 2026-06-25 |

## Automatic session memory

| Project | Approach | Downsides for this goal |
|---|---|---|
| [claude-mem](https://github.com/thedotmack/claude-mem) (116k★) | Captures everything the agent does, compresses it with an LLM and injects relevant context into later sessions | Volume over verification: an agent's wrong guess is stored the same way as a confirmed cause. No decision states |
| [agentmemory](https://github.com/rohitg00/agentmemory) (30k★) | 12 hooks record session events across many agents, with secret scrubbing; one store for all tools | Same trade-off as claude-mem, plus a memory server and hooks in every agent |
| [episodic-memory](https://github.com/obra/episodic-memory) (499★) | Semantic search over past conversation logs of Claude Code, Codex, Cursor and others | Raw history: the answer and the abandoned attempts come back together, and nothing marks which was right |
| [claude-subconscious](https://github.com/letta-ai/claude-subconscious) (2.9k★) | A background Letta agent reads every transcript and whispers guidance before each prompt | Marked by its authors as a demo, not for production. Sends every transcript to another agent |

## Why Hindsight

Memory engines such as [Mem0](https://github.com/mem0ai/mem0),
[Graphiti](https://github.com/getzep/graphiti), [Cognee](https://github.com/topoteretes/cognee),
[Supermemory](https://github.com/supermemoryai/supermemory) and
[MemPalace](https://github.com/MemPalace/mempalace) are alternatives to Hindsight
itself rather than to the skill. The skill is built on Hindsight because its workflow
relies on what Hindsight provides out of the box (checked 2026-11-13):

- `retain` with the same `document_id` replaces that document's facts, so a corrected
  entry keeps its id, and `get_document` returns the exact stored text for read-back.
  Since its April 2026 algorithm, open-source Mem0 only adds memories and never
  overwrites them.
- One PostgreSQL with pgvector holds vectors, keyword search and the entity graph.
  Graphiti needs Neo4j or FalkorDB; Mem0 keeps graph memory to its paid platform.
- The MCP server is built in, with an endpoint per bank. Graphiti's is marked
  experimental and not for production.
- Background consolidation turns facts from separate entries into observations that
  keep their evidence.

## Costs of this approach

- Recall and selective saving are initiated by the active agent without a separate
  user request. This depends on the host loading the skill and the agent following
  it; the project does not provide an independent transcript-capture process.
- A configured Hindsight service and its extraction provider are required. The
  optional Compose example in [`server/`](../server/) uses OpenRouter by default; the
  skill itself uses whichever server the user has connected over MCP.
- Entries are not reviewed in pull requests the way `ce-compound` learnings are; a
  review happens only when the user asks the agent for one.
