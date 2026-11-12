# Distilled Experience

**English** · [Русский](README.ru.md)

A skill that helps coding agents remember useful lessons: what failed, what
worked, how it was checked, and why a decision was made. Memory lives in
[Hindsight](https://github.com/vectorize-io/hindsight) and is used through MCP.

You need a Hindsight server with a bank named `distilled-experience` (you can
[run one locally](#run-hindsight-locally)) and an agent that supports HTTP MCP.
For a comparison with other memory tools, see [alternatives](docs/alternatives.md).

## Install

```bash
npx skills add akmalovaa/distilled-experience --skill distilled-experience --agent claude-code codex --global --yes
```

List your agents after `--agent`. Without `--global`, installation applies only
to the current project.

## Run Hindsight locally

Skip this if you already have a Hindsight server. The [`server/`](server/) folder
runs Hindsight 1.2.1 in Docker with its API, UI and embedded PostgreSQL; ports bind
to loopback only. The first start downloads the image and embedding models.

```bash
git clone https://github.com/akmalovaa/distilled-experience.git
cd distilled-experience/server && cp .env.example .env
```

In `.env`, fill in `LLM_API_KEY`: Hindsight sends saved entries to this LLM to
extract facts. The example uses OpenRouter; `.env.example` also shows a local
Ollama setup. Set `MEMORY_LANGUAGE=Russian` if you will write entries in Russian.
Then start the server:

```bash
docker compose up -d --wait --wait-timeout 1260
```

Create the bank with the skill's settings. `bank.json` sets the bank's missions and
turns the reranker off: on a CPU it can make each search take tens of seconds.

```bash
curl -fsS -X PUT http://127.0.0.1:8888/v1/default/banks/distilled-experience -H 'Content-Type: application/json' -d '{"name":"Distilled Experience"}'
```

```bash
curl -fsS -X PATCH http://127.0.0.1:8888/v1/default/banks/distilled-experience/config -H 'Content-Type: application/json' -d @bank.json
```

In the next step, use `http://127.0.0.1:8888` as `<server>`. The UI is at
<http://127.0.0.1:9999>. `docker compose down` stops the server; memory stays in
`server/data/`.

## Choose models

Start with `minimax/minimax-m3.1-flash`, the example's LLM. For short Russian
memories on CPU, use `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` if you enable
reranking; the bank starts with it off to keep searches fast. See
[model selection](docs/models.md) for comparisons and settings, including those
a local Gemma through Ollama needs.

## Connect

Add an HTTP MCP connection named `distilled-experience` in each agent. Use the URL
`<server>/mcp/distilled-experience/`, replacing `<server>` with your Hindsight URL.

For **Codex**, add this to `~/.codex/config.toml`, replacing the example address:

```toml
[mcp_servers.distilled-experience]
url = "https://memory.example.com/mcp/distilled-experience/"
```

For **Claude Code**, run:

```bash
claude mcp add --transport http --scope user distilled-experience https://memory.example.com/mcp/distilled-experience/
```

For **OpenCode**, run:

```bash
opencode mcp add distilled-experience --global --url https://memory.example.com/mcp/distilled-experience/
```

If your server requires authentication, configure it on the MCP connection using
the [Codex settings](https://learn.chatgpt.com/docs/extend/mcp?surface=cli) or
[Claude Code settings](https://code.claude.com/docs/en/mcp). Start a new agent
session if the connection's tools are not yet available.

Check the connection by asking the agent:

```text
Call get_bank through the distilled-experience MCP connection and confirm that
bank_id is exactly distilled-experience. Do not create or reconfigure the bank.
```

## Use during ordinary tasks

Add this instruction to `~/.codex/AGENTS.md` for Codex, `~/.claude/CLAUDE.md`
for Claude Code and `~/.config/opencode/AGENTS.md` for OpenCode:

```text
Use the installed distilled-experience skill during ordinary tasks. Consult
relevant experience before debugging or design decisions, and search again when
the cause is unclear or a fix fails. Before finishing, save or update verified
reusable lessons and decision reasons; write nothing if there is no useful lesson.
Use the distilled-experience MCP connection for all memory reads and writes,
and confirm with get_bank that bank_id is exactly distilled-experience.
```

Then give the agent your usual tasks. You do not need to say “remember” each time.
For example, after resolving a permissions error, the agent can save the symptom,
the fix and the check that confirmed it, so the next investigation starts there.

The [skill instructions](skills/distilled-experience/SKILL.md) contain the complete
workflow for searching, saving and correcting memory.

## Update

Use `npx skills update` for an installed remote skill. After editing a local clone,
refresh the installed copy by running this from the clone:

```bash
npx skills add . --skill distilled-experience --agent claude-code codex --global --yes
```

[MIT license](LICENSE). This project is independent of the authors of Hindsight.
