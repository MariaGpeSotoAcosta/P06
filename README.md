# How Much Expert Effort Does It Take to Trust an LLM Judge?

**Validating and bias-auditing an LLM-as-judge for synthetic social-engineering messages with a small expert panel**

María Guadalupe Soto Acosta, Elivier Armando Reyes Dávila, Mahdi Zaarei
Tecnologico de Monterrey, School of Engineering and Sciences, Zapopan, Mexico

Research Internship Project P06 (Small-Panel Fidelity Calibration), August–November 2026. Draft for submission to the ACL 2027 Student Research Workshop.

---

## What this project does

An LLM-as-judge can score thousands of generated messages, but it may disagree with experts or favor text from its own model family. We built a four-dimension rubric (**realism, persuasion, naturalness, expert-judged deceptiveness**, 1–7), a rating corpus with planted defects, and a judge made of four model families (Anthropic, OpenAI, Google, DeepSeek). We then measured how well the judge agrees with a small expert panel, audited its biases, and estimated how much expert effort is needed to trust it.

**Main results (final round, 100 messages):**

- Human agreement is low: Krippendorff’s α = 0.26–0.42 (two retained raters).
- The ensemble of four judges ranks messages at least as well as one expert ranks another (Spearman ρ with the panel 0.49–0.63) and meets the pre-registered decision rule on all four dimensions.
- The ensemble orders 95–97% of clean/defective pairs correctly, against 68–76% for the panel mean.
- No self-preference (−0.08 to +0.09 points) and no position bias (88–94% consistency when the order is swapped).
- Weak point: absolute calibration. Judges score persuasion 0.8–1.7 points lower and naturalness up to 1.1 points higher than the experts.
- About 60 expert-rated items reach a stable conclusion, and every added rater improves the reference more than adding more items does.

## Safety and ethics

All messages are **synthetic, fictional training specimens**: invented organizations and people, `.example` domains and 555 phone numbers. Every image shown to raters carried a “fictional training example” notice. No message was ever sent to anyone, and raters were never asked whether they would comply. Raters are identified only by anonymous codes (R1–R3).

---

## Repository layout

```
P06-llm-judge-validation/
├── README.md
├── notebooks/        Google Colab notebooks (study rounds and superseded versions)
├── data/             The 100 rated messages, their metadata and the expert ratings
├── results/          Outputs of the final analysis (main and sensitivity)
├── prompts/          The exact LLM-judge prompt used in the final round
└── paper/            Manuscript and the results report
```

### `notebooks/`

The study ran in three rounds, one Colab notebook each (`part1`–`part3`). Notebooks that were replaced by a later version are kept as `unused_nX` for transparency.

| File | Round | What it does |
|---|---|---|
| `part1_dryrun_judges_v1.ipynb` | Dry run (Sept.) | Judge prompt v1 on the 14 dry-run campaigns (7 clean/defective pairs): 4 judges × 4 dimensions × 3 runs = 672 calls, with a preflight test and recorded temperature. Prompt v1 leaked the planted defect through its context fields, so these scores are not used in the paper. |
| `part2_pilot_week9_judge_vs_panel.ipynb` | Pilot (Week 9) | 28 form positions (14 originals + paraphrases), 5 raters, 4 judges (1,344 calls): panel reliability, judge pass, judge–panel agreement and first bias checks. |
| `part3_final_judge_vs_panel.ipynb` | Final (Weeks 11–13) | **Main notebook.** 100 messages, 3 raters, 4 judges × 3 runs (4,800 calls) plus a 296-call order-swap test. Produces every number in the paper. Data are embedded in the notebook. |
| `unused_n1_judges_v1_three_models.ipynb` | Dry run | First version of part 1 with three judges only (Claude, GPT, Gemini). Replaced by part 1. |
| `unused_n2_judges_v2_power.ipynb` | Between dry run and pilot | Re-ran the four judges with prompt v2 (hint removed), calibration, self-preference and a power simulation on the 14 dry-run campaigns. Used for supervisor feedback; superseded by the pilot. |
| `unused_n3_final_v1_first_draft.ipynb` | Final | First version of part 3 (2 raters). Cached empty answers from reasoning models as done. |
| `unused_n4_final_v2_retry_fix.ipynb` | Final | Second version of part 3: larger token budget and retry of empty answers (2 raters). Replaced when the third rater arrived. |

**What `part3_final_judge_vs_panel.ipynb` contains**

| Part | Content |
|---|---|
| Configuration | Judges (`claude-sonnet-5-5`, `gpt-5.6-terra`, `gemini-3.6-flash`, `deepseek-flash`), 3 runs per item, temperature 0 when accepted, 4,000 max tokens, seeds, `MOCK` mode, `APPLY_EXCLUSIONS` (pre-registered rules on/off). |
| Resume | Upload a previous `B_judge_raw_log.jsonl`. Valid answers are reused; only missing, empty or truncated answers are asked again. |
| Data | The 100 messages and the 1,200 expert ratings, embedded. |
| A. Panel reliability | Pre-registered exclusion checks, Krippendorff’s α (ordinal and interval) with bootstrap CIs, pairwise agreement, rater severity, correlations between dimensions, clean/defective pairs (overall, by source, by planted defect), scores by generator, fatigue by section. |
| B. LLM judge | One dimension per call, rubric text identical to the rating form, fixed evaluation steps, JSON output, 3 runs with the median kept, shuffled order with a seed, full provenance log. |
| C. Agreement | Spearman, Kendall, MAE and bias with bootstrap CIs; judge vs. each rater; leave-one-out comparison on equal footing; α with the judge as an extra rater; the pre-registered decision rule. |
| D. Bias audit | Self-preference (difference-in-differences, every judge family has its own items), order-swap position test, verbosity, run-to-run stability. |
| E. Efficiency | Item subsampling (10–100 items, 1,000 draws each) and panel-size analysis (1, 2 and 3 raters). |
| F. Report | Figures, CSV tables, `key_numbers.json` and an HTML report, zipped. |

**How to run**

1. Open the notebook in Google Colab (File → Upload notebook).
2. Add your API keys in Colab → 🔑 Secrets (`ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GEMINI_API_KEY`, `DEEPSEEK_API_KEY`; older notebooks use `GOOGLE_API_KEY` for Gemini) and enable notebook access.
3. To test without API calls, set `MOCK = True` (results are meaningless in this mode).
4. Run all. To reproduce the paper exactly without new API calls, upload `results/main_R1_R2/B_judge_raw_log.jsonl` when the notebook asks: every answer is then read from the cache.
5. For the 3-rater sensitivity analysis, set `APPLY_EXCLUSIONS = False`.

Model IDs are those available to us in November 2026; check the provider documentation before running.

### `data/`

| File | Content |
|---|---|
| `images/FORM-001.png` … `FORM-100.png` | The 100 messages exactly as raters saw them in the form, in form order. |
| `items_100.csv` | One row per form position: `item_id` in the master index, source set, pair, version (clean/defective), planted defect, scenario, channel, generator family and model, temperature, seed, and the text given to the judge. |
| `index_122.csv` | Master index of the 122 rendered cards. The form used ITEM-001…040 and ITEM-063…122 in that order. |
| `panel_ratings_R1_R3.csv` | 1–7 ratings on the four dimensions and confidence of the three raters (anonymized). R3 fails pre-registered exclusion rule 2 and is used only in the sensitivity analysis. |

**Composition of the 100 messages**

| Block | Generator | Messages | Pairs | Form positions |
|---|---|---|---|---|
| Stamped scenarios | Claude (Anthropic) | 14 | 7 | 1–14 |
| SIM campaigns (teammate’s generator) | phi4-mini, llama3.1:8b, qwen3:8b | 26 | — | 15–40 |
| Set 1 | ChatGPT (OpenAI) | 20 | 10 | 41–60 |
| Set 2 | DeepSeek | 20 | 10 | 61–80 |
| Set 3 | Gemini (Google) | 20 | 10 | 81–100 |

In total, 74 messages form 37 clean/defective pairs and 26 campaigns have no counterpart, from seven generator families. Each defective version plants one defect that targets one dimension: realism (sender or format inconsistent), persuasion (no deadline or consequence), naturalness (translated-sounding grammar) or deceptiveness (gift-card, cryptocurrency or password request).

Text sources: sets 1–3 come from the generators’ JSON output (matched to the images). The 14 stamped scenarios and the 26 SIM campaigns were transcribed from the images with OCR and may contain minor errors in URLs.

### `results/`

| Folder | Content |
|---|---|
| `main_R1_R2/` | Main analysis with the pre-registered exclusions (R1 + R2). |
| `sensitivity_R1_R2_R3/` | Same analysis with all three raters. |

Each folder has:

- `P06_final_report.html`: full report with every table and figure.
- `key_numbers.json`: headline numbers.
- `tables/`: one CSV per analysis. The prefix gives the part of the notebook: A = panel, C = agreement, D = bias, E = efficiency.
- `figures/`: PNG figures used in the paper.
- `B_judge_runs_long.csv`: one row per judge call (judge, item, dimension, run, score, reasoning, model returned, temperature, presentation order).
- `B_judge_scores_median.csv`: the median of the 3 runs per judge, item and dimension.
- `B_judge_raw_log.jsonl` (main folder only): the raw log of every judge call (4,800 ratings + 296 order-swap pairs, plus the earlier empty or failed attempts that were retried) with full provenance; the notebook uses it as the cache.

### `prompts/`

`judge_prompt_judge-final-1.0.md` contains the exact system prompt, the per-dimension user prompt and the pairwise order-swap prompt (prompt hash `3e27985a4d44`).

### `paper/`

- `P06_Manuscript.docx`: the 7-page two-column manuscript.
- `P06_Results_Report_ES.docx`: the detailed results report, in Spanish.

---

## Study design at a glance

| Round | Items | Raters | Judges |
|---|---|---|---|
| Dry run (Sept.) | 14 (7 pairs) | 7 (4 complete) | 4 families × 3 runs (prompt v1, not used) |
| Pilot (Week 9) | 28 positions | 5 (1 excluded) | 4 families × 3 runs |
| Final (Weeks 11–13) | 100 (37 pairs + 26 SIM) | 3 (1 excluded) | 4 families × 3 runs + order-swap test |

The rubric, panel procedure, judge configuration, exclusion rules and decision rule were frozen in Protocol v2 before the final ratings. Deviations (fixed form order, no control or duplicate items, partial use of rubric v3, 3 raters instead of 4–5) are listed in the manuscript.

## Requirements

Google Colab, or Python 3.10+ with `pandas`, `numpy`, `scipy`, `matplotlib`, `krippendorff`, `anthropic`, `openai` and `google-genai`. The first cell of each notebook installs what is missing.

## Citation

> Soto Acosta, M. G., Reyes Dávila, E. A., & Zaarei, M. (2026). *How Much Expert Effort Does It Take to Trust an LLM Judge? Validating and Bias-Auditing an LLM-as-Judge for Synthetic Social-Engineering Messages with a Small Expert Panel.* Tecnologico de Monterrey. Manuscript in preparation.
