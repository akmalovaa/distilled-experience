# Distilled Experience

[English](README.md) · **Русский**

Скилл помогает coding-агентам запоминать полезный опыт: что не работало, что
помогло, как это проверили и почему приняли решение. Память хранится в
[Hindsight](https://github.com/vectorize-io/hindsight), доступ к ней — через MCP.

Нужны сервер Hindsight с банком `distilled-experience` (его можно
[поднять локально](#локальный-сервер-hindsight)) и агент с поддержкой HTTP MCP.
Сравнение с другими инструментами памяти — в [обзоре альтернатив](docs/alternatives.md)
(на английском).

## Установка

```bash
npx skills add akmalovaa/distilled-experience --skill distilled-experience --agent claude-code codex --global --yes
```

После `--agent` перечислите своих агентов. Без `--global` скилл установится
только в текущий проект.

## Локальный сервер Hindsight

Если сервер Hindsight у вас уже есть, пропустите этот раздел. Папка
[`server/`](server/) запускает Hindsight 1.2.1 в Docker: API, UI и встроенный
PostgreSQL, порты открыты только на loopback. Первый запуск скачивает образ и
модели эмбеддингов.

```bash
git clone https://github.com/akmalovaa/distilled-experience.git
cd distilled-experience/server && cp .env.example .env
```

В `.env` заполните `LLM_API_KEY`: Hindsight отправляет сохранённые записи в эту
LLM, чтобы извлечь из них факты. В примере — OpenRouter, в `.env.example` есть и
вариант с локальной Ollama. Если записи будут на русском, укажите
`MEMORY_LANGUAGE=Russian`. Затем запустите сервер:

```bash
docker compose up -d --wait --wait-timeout 1260
```

Создайте банк с настройками скилла. `bank.json` задаёт миссии банка и выключает
реранкер: на CPU он может растягивать каждый поиск до десятков секунд.

```bash
curl -fsS -X PUT http://127.0.0.1:8888/v1/default/banks/distilled-experience -H 'Content-Type: application/json' -d '{"name":"Distilled Experience"}'
```

```bash
curl -fsS -X PATCH http://127.0.0.1:8888/v1/default/banks/distilled-experience/config -H 'Content-Type: application/json' -d @bank.json
```

На следующем шаге используйте `http://127.0.0.1:8888` вместо `<server>`. UI —
<http://127.0.0.1:9999>. `docker compose down` останавливает сервер, память
остаётся в `server/data/`.

## Выбор моделей

Начните с `minimax/minimax-m3.1-flash` — LLM из примера. Для коротких записей
на русском с реранкером на CPU рекомендуем `cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`;
по умолчанию реранкер выключен ради скорости. Замеры, альтернативы и настройки,
в том числе для локальной Gemma через Ollama, — в [обзоре моделей](docs/models.md)
(на английском).

## Подключение

В каждом агенте добавьте HTTP MCP-подключение с именем `distilled-experience`.
Адрес: `<server>/mcp/distilled-experience/`, где `<server>` — адрес вашего Hindsight.

Для **Codex** добавьте в `~/.codex/config.toml`, заменив адрес из примера:

```toml
[mcp_servers.distilled-experience]
url = "https://memory.example.com/mcp/distilled-experience/"
```

Для **Claude Code** выполните:

```bash
claude mcp add --transport http --scope user distilled-experience https://memory.example.com/mcp/distilled-experience/
```

Для **OpenCode** выполните:

```bash
opencode mcp add distilled-experience --global --url https://memory.example.com/mcp/distilled-experience/
```

Если сервер требует авторизацию, настройте её в MCP-подключении по
[инструкции Codex](https://learn.chatgpt.com/docs/extend/mcp?surface=cli) или
[Claude Code](https://code.claude.com/docs/en/mcp). Если инструменты подключения
ещё не появились, начните новую сессию агента.

Проверьте подключение, попросив агента:

```text
Вызови get_bank через MCP-подключение distilled-experience и проверь, что bank_id
равен в точности distilled-experience. Не создавай банк и не меняй его настройки.
```

## Использование в обычных задачах

Добавьте эту инструкцию в `~/.codex/AGENTS.md` для Codex, `~/.claude/CLAUDE.md`
для Claude Code и `~/.config/opencode/AGENTS.md` для OpenCode:

```text
В обычных задачах используй установленный скилл distilled-experience. Перед
отладкой или выбором подхода обращайся к подходящему прошлому опыту; повторяй поиск,
если причина неясна или исправление не помогло. Перед завершением сохраняй или
уточняй проверенный полезный опыт и причины решений. Если нового полезного опыта
нет, ничего не записывай. Для всех чтений и записей используй MCP-подключение
distilled-experience; через get_bank проверь, что bank_id равен distilled-experience.
```

Затем давайте агенту обычные задачи. Говорить «запомни» каждый раз не нужно.
Например, исправив ошибку доступа, агент может сохранить симптом, исправление
и проверку результата, чтобы следующее расследование началось с этого опыта.

Полный порядок поиска, сохранения и исправления памяти описан в
[инструкции скилла](skills/distilled-experience/SKILL.md).

## Обновление

Для установленного удалённого скилла используйте `npx skills update`. После правок
в локальном клоне обновите установленную копию, выполнив из клона:

```bash
npx skills add . --skill distilled-experience --agent claude-code codex --global --yes
```

[Лицензия MIT](LICENSE). Проект не связан с авторами Hindsight.
