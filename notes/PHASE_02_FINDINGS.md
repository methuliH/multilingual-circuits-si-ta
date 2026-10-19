# Phase 0.2 Findings: Methods Paper (Transcoders, Attribution Graphs, Interventions)

> Notes from *Circuit Tracing: Revealing Computational Graphs in Language Models* (Anthropic, 2025). Only the transcoder, attribution-graph and intervention sections.
> Source: https://transformer-circuits.pub/2025/attribution-graphs/methods.html

**Status:** ✅ Skimmed · **Phase:** 0 (Setup), Task 2

> ⚠️ Specific numbers still to be double-checked against the original paper.

---

## 1. Transcoders: making the model readable

**Problem:** individual neurons are *polysemantic*. One neuron fires for many unrelated things, so you can't read meaning from it directly.

**Fix:** train a **cross-layer transcoder (CLT)**, a separate network that imitates the model's MLP layers using **sparse, interpretable features** (e.g. a "Texas" feature or an "antonym" feature).

| Part | What it does |
|---|---|
| **Encoder** | A feature reads the residual stream at one layer (linear + JumpReLU) |
| **Decoders** | It writes to that layer's MLP output *and every later layer* ("cross-layer") |
| **Training loss** | Reconstruction error (match the MLP outputs) + sparsity penalty (few active features) |

### Replacement model vs. local replacement model

| | Replacement model | Local replacement model |
|---|---|---|
| **How** | Swap real MLPs for the CLT | Same, plus frozen attention/normalization and **error nodes** for one specific prompt |
| **Accuracy** | Matches the real model's top token about **50%** of the time | Reproduces the original output **exactly** for that prompt |

- **Error nodes** = the part of each MLP output the transcoder couldn't explain
- Reported unexplained MLP output: about **11.5%** (18-layer model) to **21.7%** (Claude 3.5 Haiku)

---

## 2. Attribution graphs: the circuit diagram

A map of **which features caused which, ending at the output**.

### Node types

| Node | What it is |
|---|---|
| **Embedding** | Input tokens |
| **Feature** | An active transcoder feature at a token position |
| **Error** | The part the transcoder couldn't explain |
| **Output** | Candidate next tokens (logits); up to 10, covering 95% of probability |

### Key ideas

- **Edges** = the direct linear effect of one node on another
- **Why it's linear:** with attention patterns and normalization frozen, each feature's input is a linear sum of earlier features, so edges add up exactly
- **Pruning:** full graphs can have millions of edges. Default pruning cuts nodes by about **10×** while losing about **20%** of the explanatory power
- **Supernodes:** related features are **grouped by hand** (e.g. all "Texas"-ish features into one box). The paper says automated grouping wasn't good enough
- **Reading a graph:** label features from their top activating examples and output effects, then trace paths from input to output

---

## 3. Interventions: proving the graph is right

A graph is a **hypothesis**. An intervention **tests** it.

### How

- Multiply a feature's activation by a factor:
  - **−1** → suppress (the main method in the paper)
  - **large positive** → amplify / inject
- **Constrained patching:** apply the change across a range of layers, then run the rest of the model forward
- **Check:** do the downstream features and output change the way the graph predicts?

### Results reported

- Suppressing input features suppressed the downstream features, as predicted
- The graph's influence scores predicted real ablation effects reasonably well (Spearman correlation about **0.72**)
- Factual recall: ablating separate supernodes (e.g. "sport" vs. "Michael Jordan") showed **parallel paths**
- An addition example gave odd results, and the authors warn against over-reading them

### Caveats

- Attention stays **frozen** during interventions, so attention-driven effects aren't measured
- Suppressing with −1 **≠** removing a feature entirely
- Errors **build up across layers** after an intervention

### CLT vs. per-layer transcoders (10M-feature dictionaries)

| | Completeness | Replacement score | Avg. path length |
|---|---|---|---|
| **CLT** | 0.80 | **0.61** | 2.3 |
| **Per-layer transcoder** | 0.78 | **0.37** | 3.7 |

→ CLTs explain more and give **shorter, cleaner** paths.

---

## 4. Limitations to remember

- [ ] **Attention (QK) blind spot:** graphs don't explain *why* the model attends where it does (this also shows up in the French multilingual example)
- [ ] **"Dark matter":** error nodes can hide the important part of the computation
- [ ] **Inactive features and suppression** are hard to show in graphs
- [ ] **Graph complexity:** some graphs stay large and hard to read
- [ ] **Feature splitting and absorption:** features aren't always at the most interpretable level
- [ ] **Faithfulness isn't guaranteed:** the transcoder may mimic MLP outputs without using the same mechanism

---

## 5. Implications for my project

| Methods finding | What it means for me |
|---|---|
| Per-layer transcoders explain less than CLTs | **Check which transcoders my `circuit-tracer` setup loads for Gemma-2-2B.** If per-layer, expect messier graphs and more error nodes than Anthropic's figures |
| Error nodes = unexplained computation | **Compare error-node size across languages.** If Sinhala/Tamil errors are much larger than English, the transcoder may be poorly trained on those scripts. That's a confound to report, and possibly a finding |
| Supernodes are grouped by hand | Budget time for manual labelling and grouping, and document how I grouped features |
| −1 suppression, large positive injection | Use the same intervention style so results compare with both papers |
| Graphs are hypotheses | **Every claim needs an intervention**, not just a graph |
| Attention is a blind spot | Note it as a limitation, especially if language switching seems to happen through attention |

---

## 6. Open questions for Phase 1

- [ ] Does `circuit-tracer` load CLTs or per-layer transcoders for Gemma-2-2B by default? Are CLTs available?
- [ ] What pruning threshold does `circuit-tracer` use by default, and should I keep it consistent across languages?
- [ ] How does `circuit-tracer` expose interventions (API for scaling feature activations)?
- [ ] How do I measure error-node size per graph so I can compare across languages?
