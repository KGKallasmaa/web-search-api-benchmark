# VerticalRTK, 138 questions, 6 search APIs

Run: 2026-11-05, `benchmark.py --dataset verticalrtk`,
seed 42, all 138 rows of the Tako VerticalRTK fast set (commit f3f8628, 17
rows with a capture timestamp or a forecast dropped), `--workers 4`. Reader
`gpt-5.6-luna`, grader `gpt-5.6-sol`. One search per question. The reader
prompt gives today's date.

The run started at 00:27 CET. At 01:10 CET Brave hit its monthly spend cap
(HTTP 402) at 127 of 138 questions. The run continued at 07:18 CET after the
cap was raised, and it finished at 07:21 CET. The harness ran the Brave error
row again (`vrtk_fast_0012`). Thus every provider has 138 graded rows and no
error row.

## Result

cloro is fourth of six on accuracy, and it is the slowest provider by a
large margin. cloro answered 58.0% correct, against 71.7% for Exa and 68.8%
for Parallel. cloro's p50 latency is 33.06 s. SerpApi returns the same
Google results with the AI Overview at a p50 of 7.97 s on the same
questions.

cloro and SerpApi have almost the same accuracy (80 and 81 correct). Both
read Google. At 138 questions, Exa and Parallel are better than Tavily, and
Exa is better than Brave. All other pairs are inside each other's 95%
intervals, cloro and Exa included.

| Provider | Accuracy | 95% interval | Latency p50 | p90 | p99 | Reader tokens per question |
|---|---|---|---|---|---|---|
| Exa | 71.7% | 63.7-78.6 | 0.62 s | 2.53 s | 3.24 s | 4,962 |
| Parallel | 68.8% | 60.7-76.0 | 2.94 s | 3.93 s | 4.37 s | 3,073 |
| SerpApi | 58.7% | 50.4-66.6 | 7.97 s | 26.59 s | 76.81 s | 945 |
| cloro | 58.0% | 49.6-65.9 | 33.06 s | 61.78 s | 114.87 s | 983 |
| Brave | 54.3% | 46.0-62.4 | 1.22 s | 1.40 s | 1.82 s | 3,451 |
| Tavily | 50.7% | 42.5-58.9 | 2.08 s | 2.99 s | 3.77 s | 3,321 |

| Provider | Correct | Incorrect | Not attempted | Accuracy given attempted | AI Overview present |
|---|---|---|---|---|---|
| Exa | 99 | 36 | 3 | 73.3% | - |
| Parallel | 95 | 36 | 7 | 72.5% | - |
| SerpApi | 81 | 41 | 16 | 66.4% | 134/138 |
| cloro | 80 | 42 | 16 | 65.6% | 136/138 |
| Brave | 75 | 32 | 31 | 70.1% | - |
| Tavily | 70 | 41 | 27 | 63.1% | - |

## Search errors

| Provider | Error rows in the final data | Error lines in the file | HTTP retries on successful rows |
|---|---|---|---|
| cloro | 0 | 0 | 5 |
| SerpApi | 0 | 0 | 3 |
| Tavily | 0 | 0 | 0 |
| Exa | 0 | 0 | 0 |
| Parallel | 0 | 0 | 0 |
| Brave | 0 | 1 | 0 |

- The Brave error line is `vrtk_fast_0012`, `HTTPError 402: 'Payment Required'`
  (3 attempts). The resumed run replaced it with a graded row.
- cloro needed a second HTTP attempt on 5 questions (`vrtk_fast_0002`,
  `0031`, `0120`, `0126`, `0134`). The latency counts only the successful
  attempt, so the time of the failed attempt is not in the table.

## Accuracy per vertical

| Vertical | n | cloro | SerpApi | Tavily | Exa | Parallel | Brave |
|---|---|---|---|---|---|---|---|
| companies | 61 | 44 (72%) | 47 (77%) | 40 (66%) | 54 (89%) | 53 (87%) | 39 (64%) |
| government_defense | 24 | 11 (46%) | 7 (29%) | 9 (38%) | 15 (62%) | 12 (50%) | 10 (42%) |
| macro_markets | 20 | 11 (55%) | 12 (60%) | 12 (60%) | 14 (70%) | 15 (75%) | 11 (55%) |
| society_health | 20 | 8 (40%) | 8 (40%) | 3 (15%) | 8 (40%) | 8 (40%) | 7 (35%) |
| energy_climate | 9 | 5 (56%) | 5 (56%) | 5 (56%) | 5 (56%) | 5 (56%) | 4 (44%) |
| sports | 3 | 1 (33%) | 2 (67%) | 1 (33%) | 2 (67%) | 2 (67%) | 3 (100%) |
| digital_usage | 1 | 0 (0%) | 0 (0%) | 0 (0%) | 1 (100%) | 0 (0%) | 1 (100%) |

- The largest gap for cloro is in `companies`: 44 of 61, against 54 for Exa
  and 53 for Parallel. These are finance questions (margins, ratios, capital,
  deal values).
- cloro returned no Knowledge Graph panel on any of the 138 questions, and no
  finance, weather or sports box. SerpApi returned no answer box on any of
  them. Thus the Google rows depend on the AI Overview and the snippets.
- The `sports`, `digital_usage` and `energy_climate` rows have 9 or fewer
  questions. Do not read a difference there as a result.

## Possibly stale gold

30 of 138 questions have 4 or more providers graded INCORRECT. On 13 of them,
all 6 providers are INCORRECT. The VerticalRTK gold answers are recent (most
cite June to September 2026). Thus a miss here has two possible causes: the
providers find an older figure, or the gold is not the current figure. On
many rows the providers agree on one number, which is often the figure for
an earlier period. Review these rows by hand before you publish.

Without these 30 rows the order does not change:

| Provider | All 138 | Without the 30 | 95% interval without |
|---|---|---|---|
| Exa | 71.7% | 96/108 (88.9%) | 81.6-93.5 |
| Parallel | 68.8% | 92/108 (85.2%) | 77.3-90.7 |
| SerpApi | 58.7% | 80/108 (74.1%) | 65.1-81.4 |
| cloro | 58.0% | 78/108 (72.2%) | 63.1-79.8 |
| Brave | 54.3% | 72/108 (66.7%) | 57.3-74.8 |
| Tavily | 50.7% | 70/108 (64.8%) | 55.4-73.2 |

| Id | Vertical | Question | Gold | Common answer (providers) | Incorrect |
|---|---|---|---|---|---|
| vrtk_fast_0010 | companies | What was Chipotle's most recently reported operating margin? | About 15.7% (quarter ended June 2026) | 13.7% (5) | 6 |
| vrtk_fast_0027 | companies | What is Berkshire Hathaway's personal insurance expense ratio? | 14.6% (GEICO, quarter ended June 30, 2026) | 12.4% (5) | 6 |
| vrtk_fast_0032 | companies | Morgan Stanley's Tier 1 capital, what is it? | $97.2 billion (June 30, 2026) | None ($91.0 to $98.7 billion, or a ratio) | 4 |
| vrtk_fast_0033 | companies | Wells Fargo's total nonperforming assets, what's the figure? | $7.944 billion (June 30, 2026) | $7.36 to $7.4 billion (4) | 6 |
| vrtk_fast_0034 | companies | What was the transaction value of TowneBank's acquisition of Dogwood State Bank? | About $491 million (S&P Capital IQ) | $476.2 million (6) | 6 |
| vrtk_fast_0041 | government_defense | How much federal contract money was obligated to SecuriGence LLC in fiscal year 2025? | About $195.1 million | $198.1 to $198.3 million (5) | 5 |
| vrtk_fast_0042 | government_defense | Missile Defense Agency contract obligations, fiscal year 2025 | About $9.09 billion | $9.35 billion (3) | 4 |
| vrtk_fast_0046 | government_defense | How much did NASA pay out in federal contract outlays in FY2025? | About $20.0 billion ($19 to $21 billion is correct) | $15.8 billion (2) or $24.6 billion (2) | 5 |
| vrtk_fast_0049 | government_defense | How much US foreign aid was disbursed to Haiti in the latest year? | About $168.2 million (FY2026 year to date) | $337 million (3) | 5 |
| vrtk_fast_0052 | macro_markets | What did Microsoft pay H-1B computer and information research scientists on average in fiscal year 2026? | $203,899 | $197,848 (2) | 4 |
| vrtk_fast_0066 | macro_markets | What does a Big Mac cost in South Africa in US dollars? | About $4.2 (July 2026 Big Mac index) | None ($2.30 to $4.27) | 6 |
| vrtk_fast_0068 | macro_markets | How large were Brazil's commercial services imports in the latest reported year? | About $102.7 billion (2025) | None ($86.4 to $104.8 billion) | 4 |
| vrtk_fast_0071 | society_health | What is the latest motor vehicle theft rate per 100,000 in Sacramento? | About 385 (trailing 12 months, April 2026) | 421 (3) | 5 |
| vrtk_fast_0075 | society_health | What is the adult obesity rate in Arkansas per the latest data? | About 38.9% (2024) | 36.7% (5) | 5 |
| vrtk_fast_0080 | society_health | What is the median apartment rent in Des Moines according to Apartment List's recent 2026 rent reports? | About $1,050 to $1,065 (mid-2026) | $1,022 (4) | 6 |
| vrtk_fast_0081 | society_health | What is the median apartment rent in Tucson per Apartment List's latest rent report? | $1,050 (September 2026) | $1,041 (6) | 6 |
| vrtk_fast_0082 | society_health | Median home sale price in Fresno, latest month? | About $406,000 (August 2026) | None ($314,475 to $435,495) | 4 |
| vrtk_fast_0083 | society_health | What is the median home listing price in Omaha currently? | $285,000 (May 2026) | $295,000 (2) | 6 |
| vrtk_fast_0084 | energy_climate | How many active drilling rigs did Oman average in the latest reported month? | 52 (July 2026) | 41 (3) | 4 |
| vrtk_fast_0085 | energy_climate | What was Argentina's average drilling rig count in the latest month? | 47 (July 2026) | 38 (3) | 6 |
| vrtk_fast_0086 | energy_climate | How much electricity did Vietnam generate from solar in 2025? | About 23.1 TWh | 33.9 TWh (2) | 5 |
| vrtk_fast_0092 | energy_climate | How much rain does Mumbai typically get during the monsoon season? | About 80 inches (about 2,050 mm) | 2,200 to 2,300 mm (5) | 6 |
| vrtk_fast_0093 | digital_usage | What is Safari's mobile browser market share in the United States? | About 52% (50 to 54% is correct) | 44.74% (3) | 4 |
| vrtk_fast_0100 | sports | How many FIFA world ranking points does Morocco's men's national team have? | 1,804.0 (August 2026) | 1,805.34 (5) | 5 |
| vrtk_fast_0105 | society_health | Give me Donald Trump's current favorability numbers. | About 40.6% favorable (YouGov, June 2026) | 38% to 38.5% (5) | 6 |
| vrtk_fast_0108 | society_health | What percentage of Americans approve of the job Congress is doing? | About 12% | None (10% to 25%) | 6 |
| vrtk_fast_0111 | government_defense | What did the Department of Homeland Security spend in federal contract outlays in fiscal year 2025? | At least $20.4 billion ($19.5 to $25 billion is correct) | $28.3 to $28.4 billion (4) | 4 |
| vrtk_fast_0125 | government_defense | How much did the Department of the Navy obligate in federal contracts in fiscal year 2025? | About $176.4 billion | $185.6 billion (3) | 5 |
| vrtk_fast_0126 | macro_markets | What were Israel's commercial services exports in the latest reported year? | About $95.5 billion (2025) | $90.67 billion (5) | 6 |
| vrtk_fast_0131 | government_defense | How much did the US government obligate on drugs and biologicals contracts in fiscal year 2025? | About $26.3 billion | $20.76 billion (4) | 4 |

- The grader is not fully consistent on numbers near the gold. On
  `vrtk_fast_0042` (gold about $9.09 billion), Exa answered $9,351,766,977
  and got CORRECT. cloro answered $9,354,123,539 and got INCORRECT.
- On `vrtk_fast_0092` the gold is a rounded climate normal. 5 of 6 providers
  gave 2,200 to 2,300 mm, which is within about 12% of it.

## Leak check of raw/

Method: the same text match as the SimpleQA analysis. A raw response is
flagged when it contains a dataset name (`verticalrtk`, `vertical-rtk`,
`vertical_rtk`, `takodata`, `trytako`) or a dataset host
(`huggingface.co/datasets`, `kaggle.com`). This is a text match, not a review.

1 of 138 questions is flagged: `vrtk_fast_0128` (Department of Justice
active federal contracts in FY2025). Brave returned the GitHub pull request
`TakoData/VerticalRTK` #6, which shows the gold figure (22,900). Brave
answered 22,900 and got CORRECT. This is a real leak. No other provider
returned a dataset page. Without this row the order does not change: Exa
72.3%, Parallel 69.3%, SerpApi 59.1%, cloro 58.4%, Brave 54.0%, Tavily 51.1%.

## Cost of this run

The prediction uses the per-call estimates from the skill: cloro 7 credits,
SerpApi 2 searches, Exa about $0.007, 2 OpenAI calls per provider per
question. The actual values are from `http_calls` in the result files, the
raw responses and Exa's `costDollars`.

| API | Predicted | Actual |
|---|---|---|
| cloro | 138 calls, about 966 credits | 143 HTTP requests: 138 successful, 5 retried. About 966 credits for the successful calls (7 per call, not measured: the REST response has no credits field). |
| SerpApi | about 276 searches | 276 requests: 138 searches, 135 AI Overview follow-ups, 3 retries |
| Tavily | 138 credits | 138 requests, no retries |
| Exa | about $0.97 | $0.966 (sum of `costDollars`), 138 requests |
| Parallel | 138 searches | 138 requests. The cost is not in the response; check the dashboard. |
| Brave | 138 searches | 141 requests: 138 successful, 3 returned HTTP 402 (spend cap) |
| OpenAI | 1,656 calls | 828 reader and 828 grader calls. 2,309,504 reader prompt tokens. Grader tokens are not recorded. |

## Latency under load

This run is less comparable on latency than the SimpleQA 100-question run.
It ran at the same time as the FreshQA run and the SimpleQA 1,000-question
run. Each run had 4 workers, so about 12 requests were in progress at one
time across the 6 providers.

The SimpleQA 1,000 result files show the effect. Part of that run had no
other run active and part ran at the same time as this one (the time of
each call is from its raw file):

| Provider | SimpleQA p50, no other run (515 rows) | SimpleQA p50, during this run and FreshQA (375 rows) | VerticalRTK p50 |
|---|---|---|---|
| cloro | 10.97 s | 14.61 s | 33.06 s |
| SerpApi | 8.34 s | 7.04 s | 7.97 s |
| Tavily | 2.15 s | 1.77 s | 2.08 s |
| Exa | 0.72 s | 0.67 s | 0.62 s |
| Parallel | 3.06 s | 2.66 s | 2.94 s |
| Brave | 1.21 s | 1.19 s | 1.22 s |

- cloro is the only provider that became slower under the load: its SimpleQA
  p50 increased by about 33%. The rows with no other run are also from a
  different time of day (20:00 to 00:27 CET), so time of day can be a part
  of the change.
- The load does not explain all of cloro's VerticalRTK latency. In the same
  time windows, cloro's p50 was 14.61 s on SimpleQA and 33.06 s on
  VerticalRTK. SerpApi, with the same AI Overview, was 7.97 s on VerticalRTK.

## Before you publish

- Review the 30 possibly stale rows by hand. Decide for each row if the gold
  or the providers are correct.
- Remove or mark `vrtk_fast_0128`: a public copy of the dataset answers it.
- Find the cause of cloro's 33 s p50 on these questions before you show
  latency. Run cloro alone on VerticalRTK to separate the load from the
  query type.
