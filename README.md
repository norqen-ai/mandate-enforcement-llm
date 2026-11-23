# Mandate enforcement experiments

Code accompanying a study of rule enforcement and self-reported confidence in LLM trading agents.
Research evaluation only; no brokerage execution.

## Setup

From `code/`, create a Python 3.12 environment and install `requirements.txt` plus `pytest`.
Run `python -m pytest -q tests` for synthetic, offline tests.

The data is not included for licensing reasons. `build_pit_bundles.py` and `src/pit/data.py`
show how it was built. Historical data availability can change; recollection is not guaranteed
to recreate the original frozen inputs. No credentials, prices, news, logs or LLM cache are included.

## Reproduction order

1. `build_pit_bundles.py`: collect dated data and freeze evidence bundles. This original evidence
   stage uses a hosted evidence model; do not run it without authorization and provider access.
2. `run_pit_decisions.py`: generate proposals from frozen bundles for each profile and model.
3. `run_pit_riskagent.py`, then `analyze_components.py`: create baseline reviewer outputs and scores.
4. `run_finfm_arms.py`: run the bar-shown feedback arm and the `--hide-conf` control on the same proposals.
5. `score_finfm_arms.py`: score paired baseline/new arms using the frozen outcome rule.
6. `matched_conf_check.py`: compare kept confidence-raised and originally above-bar trades.
7. `make_finfm_tables.py`: produce primary tables in `../paper_finfm/tables/`.
8. `score_local_robustness.py`: validate and summarize the separate local-model sample.

The local supplement uses seed 20261123, one bundle per month and market, and all four profiles:
`python run_pit_decisions.py --models llama-3.1-8b-local --styles nl --since 2025-01-15 --per-month-region 1 --workers 1 --out logs/finfm_2027/local_robustness`.
The output must have 152 decisions over 38 bundles; smoke-test files must use a separate output path.
Run both feedback arms with `--decisions logs/finfm_2027/local_robustness --out logs/finfm_2027/local_robustness`.

## Provider controls

Set `PIT_DISABLE_OPENAI=1` to prohibit new OpenAI API calls. Set `PIT_FREE_ONLY=1` to prohibit paid
OpenRouter routes. Ollama uses localhost port 11434, with no hosted fallback. Download the desired
Ollama model first. Local cache keys include the model digest and inference configuration.
`PIT_CACHE_PATH` selects a separate SQLite response cache. Hosted calls require explicit provider
credentials; paid OpenRouter calls additionally require `PIT_SPEND_DB` and `PIT_BUDGET_USD`
(maximum 5). The ledger reserves an upper bound before each call and retains failed-call reservations.

The primary comparison and the smaller local robustness study are separate analyses.
Quantization, provider and sample differences must be disclosed when interpreting their results.
