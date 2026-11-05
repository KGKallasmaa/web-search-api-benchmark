# Search API benchmarks

Benchmarks for web search APIs, run by [cloro](https://cloro.dev). This repo
holds the harness, the prompts, the settings and the graded results of each
run. The repo is internal for now. The plan is to make it public with the
benchmark page on cloro.dev.

Providers: cloro (Google results with the AI Overview), SerpApi, Tavily, Exa,
Parallel and Brave.

Datasets:

- [SimpleQA Verified](https://huggingface.co/datasets/google/simpleqa-verified):
  1,000 short factual questions.
- [VerticalRTK](https://github.com/TakoData/VerticalRTK) fast set: 138
  questions on recent figures (companies, government, markets, health).
- [FreshQA](https://github.com/freshllms/freshqa): 358 questions, including
  false-premise questions.

## Results (2026-11)

Accuracy, one search per question. The interval is the 95% Wilson interval.

| Provider | SimpleQA Verified (1,000) | VerticalRTK (138) | FreshQA (358) |
|---|---|---|---|
| Parallel | 97.8% (96.7-98.5) | 68.8% (60.7-76.0) | 82.7% (78.4-86.2) |
| Exa | 97.3% (96.1-98.1) | 71.7% (63.7-78.6) | 83.0% (78.7-86.5) |
| cloro | 96.9% (95.6-97.8) | 58.0% (49.6-65.9) | 88.3% (84.5-91.2) |
| SerpApi | 94.7% (93.1-95.9) | 58.7% (50.4-66.6) | 88.5% (84.8-91.4) |
| Brave | 93.1% (91.4-94.5) | 54.3% (46.0-62.4) | 84.9% (80.8-88.3) |
| Tavily | 91.8% (89.9-93.3) | 50.7% (42.5-58.9) | 81.8% (77.5-85.5) |

Search latency p50 on SimpleQA Verified, from the 615 questions per provider
that ran with no other benchmark in parallel: Exa 0.74 s, Brave 1.21 s,
Tavily 2.00 s, Parallel 2.99 s, SerpApi 8.20 s, cloro 10.96 s.

The full reports, with the paired tests, the leak check, the questions with
possibly wrong gold answers and the costs, are in [`runs/`](runs):

- [`runs/simpleqa-2026-11/report.md`](runs/simpleqa-2026-11/report.md)
- [`runs/verticalrtk-2026-11/report.md`](runs/verticalrtk-2026-11/report.md)
- [`runs/freshqa-2026-11/report.md`](runs/freshqa-2026-11/report.md)

Print the results table of a run again from its result files:

```bash
python3 benchmark.py --summary --out runs/simpleqa-2026-11
```

## Method

All providers get the same setup:

1. **One search per question.** The query is the question text, unchanged.
   There is no query rewrite, no second search and no page fetch.
2. **10 results.** Each provider uses its default search mode.
3. **The same reader.** One LLM (`gpt-5.6-luna`) reads the search results and
   gives a short answer. It must use only the search results. If the results
   do not contain the answer, it replies "NOT FOUND". The reader gets at most
   30,000 characters of search results from each provider. For VerticalRTK
   and FreshQA the prompt also gives today's date, and for FreshQA it tells
   the reader to say so if the question has a false premise.
4. **The same grader.** One LLM (`gpt-5.6-sol`) grades each answer as
   CORRECT, INCORRECT or NOT_ATTEMPTED with the OpenAI simple-evals grader
   prompt ([`grader_template.txt`](grader_template.txt)), not changed.
5. **The same questions.** The run shuffles the questions with seed 42. A
   run with `--n 100` uses the first 100 questions of that order.

| Provider | Request | Text given to the reader |
|---|---|---|
| cloro | `POST /v1/monitor/google`, country US, AI Overview on | AI Overview, Knowledge Graph, organic title/link/snippet, People Also Ask answers |
| cloro-noaio | `POST /v1/monitor/google`, country US, no `include` | Organic title/link/snippet, People Also Ask answers |
| SerpApi | `engine=google`, `gl=us`, `hl=en`, `no_cache=true`; a second request for the AI Overview when Google loads it separately | AI Overview, answer box, knowledge graph, organic, related questions |
| Tavily | `search_depth=basic` | title/url/content |
| Exa | `type=auto`, highlights | title/url/highlights |
| Parallel | default mode, objective and query = the question | title/url/excerpts |
| Brave | `extra_snippets=true` | title/url/description/extra snippets |

Dropped rows:

- VerticalRTK (commit `f3f8628`): 17 of 155 rows have a capture timestamp
  or a forecast. Their answers change in hours, so 138 rows remain.
- FreshQA (April 21, 2026 version, TEST split): fast-changing rows and rows
  past their `next_review` date are dropped, so 358 rows remain.

### Metrics

- **Accuracy:** correct answers / all questions. A failed search counts as
  not attempted.
- **Accuracy given attempted:** correct / (correct + incorrect).
- **Latency:** the time of the search request, from the first byte sent to
  the last byte received. It does not include the LLM. If a provider needs
  two requests to give the full result (SerpApi with an AI Overview), the
  latency is the sum of both. If a request fails and the harness retries
  it, the latency is the time of the successful request. The retry is in
  `http_calls`.

### Latency setup

- All providers run in the same process, from the same machine, in the same
  time window.
- The calls are interleaved: question 1 goes to every provider, then
  question 2, and so on. Thus network changes during the run affect all
  providers equally.
- 4 requests run at the same time (`--workers 4`), for every provider.
- No provider cache: SerpApi gets `no_cache=true`. cloro does not cache.
- Do not run two benchmarks at the same time if you want to compare
  latency. In the 2026-11 runs, cloro's p50 went up by 52% when three runs
  ran in parallel. The other providers did not get slower.

## Run it

Python 3.10 or later. No packages to install. Set the API keys of the
providers that you want to test and `OPENAI_API_KEY`:

| Provider | Key |
|---|---|
| cloro | `CLORO_API_KEY` |
| SerpApi | `SERPAPI_KEY` |
| Tavily | `TAVILY_API_KEY` |
| Exa | `EXA_API_KEY` |
| Parallel | `PARALLEL_API_KEY` |
| Brave | `BRAVE_API_KEY` |

At cloro the keys are in Doppler. Most are in `marketing/dev`. `SERPAPI_KEY`
is in `eng-benchmark/prd` and `EXA_API_KEY` is in `sales/dev`:

```bash
SERPAPI_KEY=$(doppler secrets get SERPAPI_KEY --plain -p eng-benchmark -c prd) \
EXA_API_KEY=$(doppler secrets get EXA_API_KEY --plain -p sales -c dev) \
doppler run -p marketing -c dev -- \
python3 benchmark.py --providers cloro,serpapi,tavily,exa,parallel,brave --n 1000 --workers 4
```

Run `--n 3` first. It finds a broken key before you spend the budget.

- `--dataset simpleqa|verticalrtk|freshqa` selects the dataset (default
  `simpleqa`).
- `--search-only` measures latency with no LLM calls.
- `--out <folder>` sets the run folder. The default is
  `outputs/search-benchmark-<dataset>-<date>/`, which git ignores.

The first run downloads the dataset. Results go to
`<run folder>/results/<provider>.jsonl`. The raw response of each search
goes to `<run folder>/raw/<provider>/<question id>.json`, for the leak check.
A run that stops can continue: run the same command again. Rows with a
search or LLM error run again.

`python3 analysis/analyze.py <run folder> <verticalrtk|freshqa>` prints the
per-topic accuracy, the questions that 4 or more providers miss, and the
leak check of `raw/`.

### Cost of a full run

A full SimpleQA run (1,000 questions, 6 providers) used about 7,000 cloro
credits, 1,970 SerpApi searches (the AI Overview needs a second request on
97% of questions), 1,000 Tavily credits, $7.00 of Exa, 1,000 Parallel and
1,000 Brave requests, and 12,000 OpenAI calls (14.4 million reader prompt
tokens).

**A 402 does not stop the harness.** It retries 3 times, writes an ERROR row
and continues with the other providers. Watch the log. A Brave spend cap
stopped the 2026-11 runs this way.

## FAQ

### Why do you use only one search per question?

One search per question measures the search API, not the agent. Many
published scores let an LLM agent search many times and open pages. With
many tries, a strong agent can get a high score from a weak search API.

### Why is the Google AI Overview included for cloro and SerpApi?

The AI Overview is part of the Google results page. A user of a Google
search API gets it in the same response. We switched off the LLM answer
features that other providers sell as separate options (Tavily
`include_answer`, Exa `summary`), because they are not part of a default
search response. Without the AI Overview, cloro got 89% on the first 100
SimpleQA questions (`cloro-noaio`), against 99% with it.

### Can the reader LLM answer from its own memory?

It can, and this is a known limit of SimpleQA for search APIs. The reader
prompt tells it to use only the search results, and the same reader is used
for all providers. Thus any memory effect is the same for all providers.

### Do the search results contain the benchmark answers?

Sometimes. Some web pages copy SimpleQA questions and answers. On SimpleQA,
193 of 1,000 questions had at least one raw response that names the dataset.
With those questions removed, the order of the providers does not change.
Each report lists the leak check.

### Is this benchmark biased toward cloro?

cloro runs this benchmark and sells one of the tested APIs. To make the
result possible to check, the code, the prompts, the settings and the graded
answers are here. If a setting is not fair to a provider, change it and run
it again.

### Can I add a provider?

Yes. Add a search function to [`benchmark.py`](benchmark.py) and add it to
`PROVIDERS`. The function must return the raw response and a list of text
blocks for the reader.
