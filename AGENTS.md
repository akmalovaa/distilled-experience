# Working on Distilled Experience

The product is a portable skill for an existing Hindsight MCP connection.
Keep its complete workflow in `skills/distilled-experience/SKILL.md`; installation
must not depend on this repository, a shell client or a particular server deployment.

`server/` is an optional local Hindsight for users without a server: keep its image
pinned, ports on loopback and `bank.json` in step with the README's bank setup.
`docs/alternatives.md` compares other approaches; keep its check date current.

Call what the skill saves experience or an entry (опыт, запись), never a lesson
or урок, in every language and file.

Keep the README translations short and consistent. Validate skill metadata, links
and realistic agent behavior after changes; use mock responses for write scenarios.
Never put test fixtures in working memory or delete/migrate its records for testing.
Keep secrets, actual memory contents and runtime data out of Git.
