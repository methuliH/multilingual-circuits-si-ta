# Phase 1: Feature Labels (English / French antonym circuit)

**Model:** Gemma-2-2B · **Transcoders:** Gemma Scope per-layer (GEMMASCOPE-TRANSCODER-16K)
**Tool:** Neuronpedia circuit tracer (default pruning: 80% of influence)
**Notebook (code route, not yet run):** [`01_trace_en_fr.ipynb`](../notebooks/01_trace_en_fr.ipynb)

## Graphs

| Run | Language | Prompt | Neuronpedia graph |
|---|---|---|---|
| 2 ✅ | English | `The opposite of "small" is "` | [theoppositeofsma-1792985373264](https://www.neuronpedia.org/gemma-2-2b/graph?slug=theoppositeofsma-1792985373264) |
| 2 ✅ | French | `Le contraire de "petit" est "` | [lecontrairedepet-1792985445354](https://www.neuronpedia.org/gemma-2-2b/graph?slug=lecontrairedepet-1792985445354) |
| 1 | English | `The opposite of "small" is` (no final quote) | [theoppositeofsma-1792484381649](https://www.neuronpedia.org/gemma-2-2b/graph?slug=theoppositeofsma-1792484381649) |
| 1 | French | `Le contraire de "petit" est` (no final quote) | [lecontrairedepetit-1792484482417](https://www.neuronpedia.org/gemma-2-2b/graph?slug=lecontrairedepetit-1792484482417) |

Run 2 uses the paper's exact prompts. Run 1 is kept for comparison.

## Top predictions

| Run | Language | Top outputs |
|---|---|---|
| 2 | English | bigger, large, big, … |
| 2 | French | grand, grande, Grand, gros, … (" grand" alone p = 0.032; probability is split across spellings) |
| 1 | English | " not" (p = 0.118), what, large, big |
| 1 | French | …, de, un, ., grand, bien, le, donc |

**Finding:** the final opening quote matters. Without it, the model continues the sentence ("is not…", "est de…"). With it, the antonym family dominates the outputs in both languages.

## Graph scores

| Run | Language | Replacement score | Completeness score |
|---|---|---|---|
| 2 | English | 0.6236 | 0.8845 |
| 2 | French | 0.5354 | 0.8619 |
| 1 | English | 0.6262 | 0.8796 |
| 1 | French | 0.5934 | 0.8726 |

**Finding:** the transcoders explain the French graph less well than the English one (replacement 0.54 vs 0.62 in run 2). This fits the English-default picture, and it's the baseline for comparing Sinhala and Tamil.

## Shared features (middle layers, final token)

Candidates for the language-independent **antonym** and **large/big** concepts.

| Layer | Feature # | Label | Evidence (top activating examples) | In EN? | In FR? |
|---|---|---|---|---|---|
| | | *to do: trace from the top output* | | | |

## Language-specific features

### Early layers (input side)

| Layer | Feature # | Language | Label | Evidence |
|---|---|---|---|---|
| | | | *to do* | |

### Late layers (output side)

| Layer | Feature # | Graph | Neuronpedia label | Evidence | Notes |
|---|---|---|---|---|---|
| 25 | 13416 | FR, run 1 | Foreign language | Activation 66.5. Positive logits: *démocr*, *religieuses*, *supérieurs*, *supérieures*, *sauvages* | Pushes French word endings: a French output-side feature |
| 25 | 7089 | EN, run 2 | French words | Activation 77.5. Positive logits: *varandra*, *genoux*, *blessés*, *nuages*, *déchir*. Top activations are on code (`configurator`, `route`, `set_folder_status`) | Active in the **English** graph. The auto-label may be misleading: check whether it really is French-specific |

## Operand features (`small` / `petit` token)

| Layer | Feature # | Label | Shared? | Evidence |
|---|---|---|---|---|
| | | *to do* | | |

## Intervention results (Steer)

| Features steered | EN before → after | FR before → after | Random control |
|---|---|---|---|
| | | | |

## Conclusion so far

- **Prompts:** run 2 reproduces the paper's behaviour; the antonym is the top answer in both languages.
- **Scores:** French is explained less well than English by the transcoders.
- **Shared concept space replicated?** Not yet tested. Next step is finding the same feature in both graphs.
