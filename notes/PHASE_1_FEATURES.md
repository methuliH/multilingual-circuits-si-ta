# Phase 1: Feature Labels (English / French antonym circuit)

**Model:** Gemma-2-2B · **Transcoders:** Gemma Scope per-layer (GEMMASCOPE-TRANSCODER-16K)
**Tool:** Neuronpedia circuit tracer (node threshold 0.8, edge threshold 0.85, max 5000 feature nodes)
**Method:** graph JSON downloaded from Neuronpedia and compared feature by feature (same layer + index = same feature). Feature descriptions are Neuronpedia's auto-interpretations.

## Graphs

| Run | Language | Prompt | Neuronpedia graph |
|---|---|---|---|
| 2 ✅ | English | `The opposite of "small" is "` | [theoppositeofsma-1792985373264](https://www.neuronpedia.org/gemma-2-2b/graph?slug=theoppositeofsma-1792985373264) |
| 2 ✅ | French | `Le contraire de "petit" est "` | [lecontrairedepet-1792985445354](https://www.neuronpedia.org/gemma-2-2b/graph?slug=lecontrairedepet-1792985445354) |
| 1 | English | `The opposite of "small" is` (no final quote) | [theoppositeofsma-1792484381649](https://www.neuronpedia.org/gemma-2-2b/graph?slug=theoppositeofsma-1792484381649) |
| 1 | French | `Le contraire de "petit" est` (no final quote) | [lecontrairedepetit-1792484482417](https://www.neuronpedia.org/gemma-2-2b/graph?slug=lecontrairedepetit-1792484482417) |

Run 2 uses the paper's exact prompts. Run 1 is kept for comparison. Everything below is run 2 unless stated.

## Top predictions

| Language | Outputs (p) |
|---|---|
| English | large 0.406 · big 0.406 · small 0.055 · great 0.023 · huge 0.020 · `<strong>` 0.007 · not 0.007 |
| French | grand 0.816 · gros 0.059 · " grand" 0.032 · Grand 0.012 · grande 0.010 |
| English, run 1 | " not" 0.118, what, large, big |
| French, run 1 | …, de, un, ., grand, bien, le, donc |

**Findings**
- With the paper's prompts, Gemma-2-2B gives the correct antonym in both languages: ~0.85 for large/big/huge/great in English, ~0.93 for grand/gros in French.
- In English, 5.5% of the probability goes to repeating **"small"**. French doesn't show this.
- The final opening quote matters: without it, the model continues the sentence ("is not…", "est de…").

## Graph scores

| Run | Language | Replacement | Completeness | Feature nodes | Error nodes |
|---|---|---|---|---|---|
| 2 | English | 0.6236 | 0.8845 | 1085 | 138 |
| 2 | French | 0.5354 | 0.8619 | 1364 | 129 |
| 1 | English | 0.6262 | 0.8796 | | |
| 1 | French | 0.5934 | 0.8726 | | |

**Finding:** the transcoders explain French less well than English (replacement 0.54 vs 0.62), and the French graph needs more features. This is the baseline for Sinhala and Tamil.

## Overlap at the final token, by layer

Same feature (layer + index) active at the final `"` token in both graphs. IoU = shared / union.

| Layers | EN features | FR features | IoU | Reading |
|---|---|---|---|---|
| 0–7 | 78 | 86 | 0.17–0.55 | High, but both prompts end in the same `"` token, so this reflects the token, not meaning |
| 8–19 | 76 | 74 | 0.13–**0.90** (L8 0.83, L17 0.90, L19 0.58) | **Highest sharing**: the middle layers |
| 20–25 | 88 | 199 | **0.10–0.25** | Language-specific: French needs ~2× as many late-layer features |

Overall at the final token: 239 EN features, 354 FR, **128 shared** (IoU 0.28). No unrelated-prompt baseline yet.

## Shared features: the "large / size" concept (final token)

Active in **both** languages, ranked by importance in the pruned graph.

| Layer | Index | Neuronpedia description | Top positive logits | Label |
|---|---|---|---|---|
| 18 | 9402 | mentions of companies and their size | size, Size, sizes, sized | **size** |
| 19 | 8658 | "large size" | Huge, gigantic, size, huge | **large** |
| 20 | 9765 | words that describe intensity, size, or level | (noisy) | size / intensity |
| 16 | 7299 | adjectives and nouns related to measurement and size | (noisy) | size |
| 21 | 10062 | adjectives indicating size | **large, huge, massive** | **say large** |
| 23 | 8683 | the words "too" and "large", related size words | **large, Large, larger, big** | **say large** |
| 23 | 9338 | subjective adjectives of size and scale | lar-, lark | size |
| 19 | 5058 | "too", "sufficiently" | (noisy) | qualifier |

**Notice:** the shared "say large" features (L21 #10062, L23 #8683) push **English** words (*large, huge, big*) even in the French graph. That fits the paper's **English-as-default** finding.

## Shared features on the operand (`small` / `petit` token)

| Layer | Index | Neuronpedia description | Top positive logits |
|---|---|---|---|
| 3 | 12834 | the word "small" and words associated with size | (noisy) |
| 6 | 4362 | terms related to size | size, sized, Size |
| 8 | 5268 | small, personal devices | diminutive, miniature, Smaller, **réduite** |
| 15 | 5756 | adjectives describing size or quantity; "big" | size, sized, Massive |

65 features are shared on this token. The meaning of "small" / "petit" is represented by the same features in both languages, as the paper found.

## Language-specific features (late layers, final token)

### French only

| Layer | Index | Neuronpedia description | Top positive logits |
|---|---|---|---|
| 20 | 1454 | French text, conversational French | plus, non, la, é |
| 22 | 10566 | French words, forum/chat text | dans, pour, sur, et, des |
| 23 | 2592 | French language | (old-spelling English) |
| 24 | 9342 | mix of French and English | (noisy) |
| 25 | 13416 | Foreign language (run 1) | démocr, religieuses, supérieurs |

L20 #1454 is the same feature that the official `circuit-tracer` intervention demo uses as its "French" feature.

### English only

| Layer | Index | Neuronpedia description | Top positive logits |
|---|---|---|---|
| 23 | 14407 | words that have a root meaning of "big" | (noisy) |
| 20 | 3830 | size, parameters | Smaller, smaller, Small, larger |
| 23 | 1343 | relative comparisons of size/quantity | (noisy) |
| 25 | 9600 | words related to size and comparison | (noisy) |

English has more **direct size/comparison** features at the output, while French routes through **French-language** features.

### Shared syntax features (not meaning)

L17 #4143 quotes, L23 #14350 quotation marks/parentheses, L24 #4707 code snippets. These come from the quote marks in both prompts.

## Not found

- No feature clearly labelled **"antonym / opposite"** among the top shared features. The operation may be spread across many features, sit in attention (invisible to this method), or the auto-labels may miss it.

## Intervention results (Steer)

*Not done yet.* Suggested test: steer **L21 #10062** and **L23 #8683** (shared "say large") negative in both graphs, and steer one random feature as a control.

## Conclusion so far

- ✅ **Shared concept replicated:** the **size / "large"** concept and the meaning of **small/petit** use the same features in English and French, concentrated in the **middle layers**.
- ✅ **Language-specific edges replicated:** late layers are mostly language-specific, with French-text features only in the French graph.
- ✅ **English default visible:** shared "say large" features push English words even in French.
- ❓ **Antonym operation:** not found as a single shared feature.
- ⚠️ **Caveats:** one prompt pair, no unrelated baseline, auto-generated labels, no intervention yet.
