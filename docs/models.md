# Models

## LLM

Start with **MiniMax M3.1 Flash** (`minimax/minimax-m3.1-flash`) for speed and
low cost. Choose **Gemini 3.8 Flash** (`google/gemini-3.8-flash`) if preserving
details when merging memories matters more. **Gemma 4 26B**
(`gemma4:26b-a4b-it-qat`) is the local option through Ollama; it needs
[its own settings](#gemma-through-ollama).

We tested the same 17 synthetic Russian scenarios twice with each model:

| Check | MiniMax M3.1 Flash | Gemini 3.8 Flash | Gemma 4 26B |
|---|---:|---:|---:|
| Extracting facts | 16/16 | 16/16 | 11/16 |
| Merging memories | 8/10 | 10/10 | 8/10 |
| Answering from supplied facts | 6/8 | 6/8 | 8/8 |
| Total fully passing answers | 30/34 | 32/34 | 27/34 |
| Valid JSON and schema, structured answers | 26/26 | 26/26 | 26/26 |
| Median response time | 1.9 s | 14.9 s | 33.4 s |
| API cost for 34 requests | $0.016 | $0.377 | Local |

MiniMax twice kept an old preference next to the current one in the present
tense, without its date, and twice added unrelated CI-cache details to an answer.
Gemini also added the CI-cache details, but kept old and current preferences separate.

MiniMax was tested on November 13, 2026; Gemini and Gemma on November 3.
The prompts and grading criteria were the same; sampling settings differed.
Gemma used `temperature=1.0`, thinking enabled and `num_ctx=65536`.
MiniMax and Gemini used temperatures of `0.1` for extraction, `0` for merging
and `0.9` for answering, with `reasoning_effort=high`.
The models received prompts from Hindsight 1.1.0 directly; this was not an
end-to-end benchmark, and `server/` runs Hindsight 1.2.1.
Timing is approximate because the runs were on different days. Search was not tested.

### Gemma through Ollama

Hindsight 1.2.1 sends Ollama the temperatures MiniMax and Gemini used above, with
thinking turned off. Gemma's results come from different settings: at these low
temperatures with thinking on, it twice used up its 16,384-token output limit
without an answer. With Hindsight's own settings, an end-to-end check on
November 12 saved every entry in 5–14 s, but left one unconfirmed hypothesis out
of the extracted facts.

To use the tested settings, add these to the Ollama block in `server/.env`:

```bash
OLLAMA_NUM_CTX=65536
LLM_EXTRA_BODY='{"think":true,"options":{"temperature":1.0}}'
```

These options override Hindsight's temperature for every call. A mocked request
in the 1.2.1 image confirmed the payload; the setting was not run against a model.

## Reranker for Russian

For short memories in Russian on CPU, start with
**[mMiniLMv2](https://huggingface.co/cross-encoder/mmarco-mMiniLMv2-L12-H384-v1)**
(`cross-encoder/mmarco-mMiniLMv2-L12-H384-v1`). It is small and supports Russian;
Hindsight's default, `hindsight-ai/hs-rerank-1`, does not list Russian among its
languages. The [input limit of mMiniLMv2 is 512 tokens](https://huggingface.co/cross-encoder/mmarco-mMiniLMv2-L12-H384-v1/blob/main/tokenizer_config.json)
for the query and passage together; longer memories need separate evaluation.

On November 12, 2026, inside the Hindsight 1.2.1 container, we scored 32 test
observations for each of 12 Russian questions on a CPU with 10 cores. The times
below cover reranking only, not the full `recall` request:

| Model | Correct result first | Reranking time per question |
|---|---:|---:|
| mMiniLMv2 | 12/12 | 0.5 s |
| BGE reranker v2 M3 | 12/12 | 5.7 s |
| hs-rerank-1 | 11/12 | 0.4 s |

mMiniLMv2 matched BGE's first-place accuracy and was much faster in this test.
Reranking time grows with the number of candidates, and `recall` reranks up to 300
by default. On a test bank of 12 entries, a full `recall` took about 3 s with
mMiniLMv2 and 0.05–0.1 s without reranking. On another server, a full `recall` with
`hs-rerank-1` took 1.4 s at 30 candidates and 19–23 s at 248–300. This small test
does not establish quality or speed on larger banks or with long memories.

Set `RERANKER_MODEL=cross-encoder/mmarco-mMiniLMv2-L12-H384-v1` in `server/.env`;
other deployments set `HINDSIGHT_API_RERANKER_LOCAL_MODEL`. From `server/`, recreate
the container with the new setting; the model, about 0.5 GB, downloads on first start:

```bash
docker compose up -d --wait --wait-timeout 1260
```

`docker compose restart` would keep the old environment. Then enable reranking for
the bank, which this example creates with it off:

```bash
curl -fsS -X PATCH http://127.0.0.1:8888/v1/default/banks/distilled-experience/config \
  -H 'Content-Type: application/json' \
  -d '{"updates":{"enable_reranking":true}}'
```

Use your server's URL and API port if they differ from the local defaults.
