# Running and inspecting the solution

Run the README commands from the unpacked archive root. Complete replay uses saved inputs, not sibling projects, model credentials or downloaded embeddings.

## Stage outputs

| Artifact | Contents |
|---|---|
| data/reviews.jsonl | Working reviews with zero-based source row IDs |
| data/themes.json | The supplied theme hierarchy |
| responses/accepted_B.jsonl | Validated saved extraction answers |
| outputs/full_grouping/candidates.json | Original embedding candidates |
| responses/F_completed_partition.json | Complete model-reviewed partition |
| responses/api_review/ | Saved requests, responses and structural corrections |
| responses/final_editorial_corrections.json | One wording correction, separate from model answers |
| outputs/submission/extraction/ | Extractions, aspects, coverage and provenance |
| outputs/submission/insights.json | Descriptions and exact membership |
| outputs/submission/mapping.json | Lexical themes with supporting aspects and review IDs |
| outputs/submission/theme_counts.json | Unique supporting reviews per theme |
| outputs/submission/category_counts.json | Review-ID union per category |
| outputs/submission/evaluation.json | Coverage, methods, hashes, limitations and cost counters |
| docs/examples.json | Five selected cases, complete reviews and input hashes |

## Optional candidate regeneration

~~~bash
.venv/bin/chattermill replay --answers responses/accepted_B.jsonl --out outputs/full_replay
.venv/bin/chattermill cluster --full --aspects outputs/full_replay/aspects.jsonl --out outputs/recomputed_candidates
~~~

This downloads or uses cached intfloat/e5-small-v2 at revision ffb93f3bd4047442299a41ebb6fa998a38507c52. It requires compute and model access. It does not regenerate LLM answers. The saved final partition is tied to its original candidates; do not overwrite them and reuse an unrelated partition.

To rebuild the selected-case appendix from current submission outputs:

~~~bash
.venv/bin/python -m cm.presentation
~~~

## Experiment evidence

responses/legacy_*.jsonl contains the 50-review extraction comparison. outputs/grouping/grouping_summary.json contains the distance sweep; its planned 300-review comparison was not completed. outputs/retrieval/ contains frozen queries, results and overlap diagnostics. outputs/dense_mapping_rejected.json and outputs/dense_theme_counts_rejected.json retain the rejected dense mapping. responses/A_corpus_findings.json contains exploratory patterns, not exhaustive memberships.

Historical reports describe their own stage. Preparation-stage zero-credit metadata does not supersede the final evaluation or provider budget status.

## Fresh generation is different from replay

cm.api_review and cm.api_mapping contain optional runners and format checks. Fresh extraction is not a one-command API stage; it was generated in browser Pro. API regeneration requires separately authorized credit and ANTHROPIC_API_KEY for the compatible proxy. Challenge credit is exhausted: do not retry the original key. Full compact model mapping inference was not run; schema tests are not a completed experiment.

The complete replay makes no model requests. Installing dependencies or optionally downloading weights is a separate network dependency, not an LLM API call.
