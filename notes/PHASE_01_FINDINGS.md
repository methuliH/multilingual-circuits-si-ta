# Phase 0.1 Findings: Multilingual Circuits (Biology Paper)

> Notes from the **Multilingual Circuits** section of *On the Biology of a Large Language Model* (Anthropic, 2025).
> Source: https://transformer-circuits.pub/2025/attribution-graphs/biology.html

**Status:** ✅ Read · **Phase:** 0 (Setup), Task 1

> ⚠️ Specific numbers (overlap counts, intervention strengths) still to be double-checked against the original paper.

---

## 1. Setup

| | |
|---|---|
| **Model** | Claude 3.5 Haiku (compared against a smaller model for the cross-language analysis) |
| **Task** | Antonym completion |
| **Method** | Attribution graphs + feature interventions |

### Exact prompts

| Language | Prompt | Expected output |
|---|---|---|
| English | `The opposite of "small" is "` | big |
| French | `Le contraire de "petit" est "` | grand |
| Chinese | `"小"的反义词是"` | 大 |

---

## 2. Claims

### Claim 1: The computation splits into three independent parts

| Component | What it does | Shared or language-specific? |
|---|---|---|
| **Operation** | Recognizes the task as "antonym" | Shared (middle layers, final token) |
| **Operand** | Represents the meaning of "small" (its size facet) | Shared (early features on the "small" token) |
| **Language** | Detects the input language and chooses the output word | Language-specific (e.g. "open quote in French") |

Antonym features act through attention to map "small" → "large". Separately, language features select which word for "large" gets said.

### Claim 2: Interventions confirm each component

| Intervention | How | Result |
|---|---|---|
| **Swap the operation** | Suppress antonym features; inject synonym features taken from `A synonym of "small" is "` | Outputs a **synonym** of small, in the **correct language** |
| **Swap the operand** | Replace "small" features with "hot" features from an English prompt, at the small/petit/小 token | Outputs an **antonym of hot** (cold), in the **correct language** |
| **Swap the language** | Replace one language's detection features with another's | **Output language changes**; operation and operand are preserved |

🔑 **Key point:** features taken from **English** prompts work inside **French and Chinese** prompts. This is the strongest evidence for a shared concept space.

### Claim 3: Sharing peaks in the middle layers

- **Metric:** intersection-over-union (IoU) of active features between a paragraph and its translation, per layer
- **Baseline:** IoU between *unrelated* paragraphs in the same language pair
- **Result:**
  - Early layers → language-specific (tokens → meaning)
  - **Middle layers → most language-agnostic**
  - Late layers → language-specific (meaning → tokens)

### Claim 4: Bigger models share more

- Claude 3.5 Haiku shares more than the smaller model
- The gain is **largest for language pairs without a shared alphabet** (English–Chinese, French–Chinese)

### Claim 5: English is the default

- Shared features have **stronger direct connections to English outputs**
- Non-English outputs rely more on extra "say X in language Y" features
- English quote features suppress the features that would otherwise suppress "large" in English, while promoting "large" in other languages
- In their words, it's a multilingual representation **with English as the default output**

### Other details

- Multi-token words (e.g. "contraire") are **detokenized** into abstract multilingual features in early layers
- Part of the antonym → "large" step works by changing **where attention looks**. This is invisible to their method, which they acknowledge as a limitation.

---

## 3. Caveats they admit

- [ ] Overlap isn't perfect: by their count, most multilingual features are active on all three prompts, but far fewer survive into all three pruned graphs
- [ ] Some interventions needed **unnaturally strong** activations (several times normal)
- [ ] Attention-pattern effects are **not captured** by the method
- [ ] Diagrams are **heavily simplified**
- [ ] Feature labels are **rough and subjective**

---

## 4. Implications for my project

| Their finding | What I'll do |
|---|---|
| Sharing is weakest for languages **without a shared alphabet** and in **smaller models** | Test the hardest case: **Sinhala + Tamil** (own scripts) on **Gemma-2-2B** (small) |
| Middle layers share, edges don't | Check whether Sinhala/Tamil show the same middle-layer peak or a flat curve |
| Three interventions | Replicate **all three** with Sinhala/Tamil as targets |
| English is the default | Check whether Sinhala/Tamil depend even more on "say X in language Y" features, or **leak English answers** |
| IoU on translated paragraphs vs. a baseline | Use **their exact metric** so results compare directly |

---

## 5. My hypotheses

- **H1:** Middle-layer feature overlap (IoU) with English will be **lower** for Sinhala and Tamil than for French.
- **H2:** The middle-layer peak will be **weaker or missing** for Sinhala and Tamil.
- **H3:** Injecting English-derived operand features ("hot") will change the Sinhala/Tamil output **less reliably** than the French output.
- **H4:** Gemma-2-2B will show a **stronger English default** for Sinhala/Tamil, e.g. sometimes answering in English.
- **H5 (optional):** Transliterated Sinhala (`loku`) will overlap with English **more** than Sinhala script (`ලොකු`).

**Write-up framing:**
> Anthropic found that concept sharing is weakest for languages with different scripts and in smaller models. I tested the hardest case: a 2B model on Sinhala and Tamil.

---

## 6. Open questions to resolve in Phase 1–2

- [ ] Which model has the best transcoder support in `circuit-tracer` for non-Latin scripts?
- [ ] How many tokens do Sinhala/Tamil antonym words take in Gemma's tokenizer?
- [ ] Does Gemma-2-2B answer the Sinhala/Tamil prompts correctly at all?
- [ ] What intervention strengths will be needed (their caveat suggests several times normal)?
- [ ] Where can I get good Sinhala/Tamil paragraph translations for the IoU analysis? (Native-speaker check needed)
