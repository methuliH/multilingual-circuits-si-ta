# Looking Inside a Mind Made of Numbers

*Part 1 of tracing multilingual circuits in Gemma-2-2B, from English and French towards Sinhala and Tamil*

---

## We know how to grow a plant 🌱

Plant a seed, add water and sunlight, and it grows. But for a long time, nobody could tell you what was actually happening *inside* it.

AI is the same. We know the architecture. We know how to train it, how to "grow" it. But what happens inside, how it actually arrives at an answer, is still mostly a mystery.

A short reel introduced me to the field trying to answer that question: **mechanistic interpretability**. It's like opening up a mind made entirely of numbers and trying to read its thoughts. I couldn't stop thinking about it.

## The paper that hooked me

I started with Anthropic's [*On the Biology of a Large Language Model*](https://transformer-circuits.pub/2025/attribution-graphs/biology.html). The researchers build **attribution graphs**: maps of which internal features cause which, all the way from the input words to the model's answer.

One section stuck with me. They asked Claude the same question in three languages:

- `The opposite of "small" is "` → *big*
- `Le contraire de "petit" est "` → *grand*
- `"小"的反义词是"` → *大*

Inside the model, the three questions ran through **the same concepts** in the middle layers: an "opposite" operation applied to a "small" concept. Only at the edges did language-specific features take over, reading the input language and choosing the output word.

They also noticed two things that became my real question:

1. This sharing is **weaker in smaller models**.
2. It's weakest between **languages that don't share an alphabet**.

Sinhala and Tamil each have their own script, and they're low-resource languages. Nobody had looked at them. But first, I needed to see the basic result myself.

## The humbling part

My laptop doesn't have a GPU. The plan was to run Anthropic's open-source [`circuit-tracer`](https://github.com/safety-research/circuit-tracer) library on Google's open model **Gemma-2-2B** in a free cloud notebook.

- **Colab** quietly ran on a CPU and crashed.
- **Kaggle** got further, then ran out of memory while loading the model.
- Then it ran out of memory again. And again.

I nearly gave up.

Then I switched to [**Neuronpedia**](https://www.neuronpedia.org/gemma-2-2b/graph), which runs the same circuit-tracing tools in the browser. No GPU, no crashes. Within minutes I had my first graph.

**Lesson learned:** the first prompts I typed were missing the final `"`. Without it, the model didn't answer the question at all. It just continued the sentence: "is *not*…", "est *de*…". Exact prompts matter.

## The experiment

| | Prompt | Gemma-2-2B's answer |
|---|---|---|
| 🇬🇧 | `The opposite of "small" is "` | **large** 41% · **big** 41% · huge · great (≈85% total) |
| 🇫🇷 | `Le contraire de "petit" est "` | **grand** 82% · **gros** 6% (≈93% total) |

Both answers are right. Then the interesting part: what happened inside?

I downloaded both attribution graphs and compared them **feature by feature**. If the same feature (same layer, same index) appears in both graphs, the two languages are sharing it.

![Feature overlap between English and French by layer](../figures/01_overlap_by_layer_en_fr.svg)

*The share of features active in both languages at the final token, layer by layer. Gemma-2-2B has 26 layers.*

## What I found

### 1. The same "size" concept in the middle 🔵

In the middle layers, the same features lit up for both languages:

| Layer | Feature | What it represents | Pushes the model to say |
|---|---|---|---|
| 18 | [#9402](https://www.neuronpedia.org/gemma-2-2b/18-gemmascope-transcoder-16k/9402) | size | *size, sizes, sized* |
| 19 | [#8658](https://www.neuronpedia.org/gemma-2-2b/19-gemmascope-transcoder-16k/8658) | large size | *huge, gigantic* |
| 21 | [#10062](https://www.neuronpedia.org/gemma-2-2b/21-gemmascope-transcoder-16k/10062) | adjectives indicating size | *large, huge, massive* |
| 23 | [#8683](https://www.neuronpedia.org/gemma-2-2b/23-gemmascope-transcoder-16k/8683) | "too" / "large" | *large, larger, big* |

The words themselves share too. "small" and "petit" activate 65 of the same features, including one that responds to *diminutive*, *miniature* and the French word *réduite*.

**Same concept, different words.**

### 2. Language takes over at the end 🟠

In the last few layers, overlap drops to about 10%. The French graph brings in features that only exist for French:

- [L20 #1454](https://www.neuronpedia.org/gemma-2-2b/20-gemmascope-transcoder-16k/1454): "French text" (pushes *plus, non, la*)
- [L22 #10566](https://www.neuronpedia.org/gemma-2-2b/22-gemmascope-transcoder-16k/10566): "French words" (pushes *dans, pour, sur*)

French also needs about **twice as many late-layer features** as English (199 vs 88). Choosing a French word seems to take more work.

A nice confirmation: L20 #1454 is the same feature that the official `circuit-tracer` demo uses as its "French" feature.

### 3. English is the default 🇬🇧

This surprised me most. The shared "say large" features (L21 #10062 and L23 #8683) push **English** words like *large* and *huge*, even inside the French graph. The French answer only appears once the French-specific features step in.

Anthropic reported the same thing in Claude: a multilingual representation with **English as the default** output.

### 4. French is harder to explain

Attribution graphs rely on **transcoders**, small networks that translate the model's internals into readable features. Whatever they can't explain becomes "error". The **replacement score** measures how much of the computation the readable features account for:

| | Replacement score | Completeness score |
|---|---|---|
| English | **0.62** | 0.88 |
| French | **0.54** | 0.86 |

The tools explain French less well than English. If Sinhala and Tamil score even lower, that tells us something about how well these tools, and maybe the model, cover those languages.

## What I didn't find

I expected a clear **"opposite / antonym"** feature shared by both languages, like the one in Anthropic's paper. It didn't show up among the top shared features. It might be spread across many weaker features, happen inside attention (which attribution graphs can't see), or just be missed by the automatic labels.

## How much to trust this

- **One prompt pair.** These are first observations, not general claims.
- **No baseline yet.** Both prompts end in the same `"` token, which inflates the early-layer overlap. Comparing against unrelated prompts would fix this.
- **Automatic labels.** Feature descriptions come from Neuronpedia's auto-interpretation, which can be misleading. One feature labelled "French words" mostly fired on code.
- **Observational.** I haven't yet **steered** these features to test cause and effect. That's the next step.

## The next seed 🌱🇱🇰

The paper's own findings point to where sharing should struggle most: small models and languages with their own scripts. Gemma-2-2B is small. Sinhala and Tamil have their own scripts.

So the next questions are:

1. Can Gemma-2-2B even answer *"the opposite of small"* in Sinhala and Tamil?
2. If it can, does it reuse the same middle-layer "size" features, or take a different route?
3. Do the transcoders explain Sinhala and Tamil as well as English, or worse than French?
4. Does the English default get even stronger?

That's Part 2.

---

**Explore it yourself:** [English graph](https://www.neuronpedia.org/gemma-2-2b/graph?slug=theoppositeofsma-1792985373264) · [French graph](https://www.neuronpedia.org/gemma-2-2b/graph?slug=lecontrairedepet-1792985445354) · [Full results](../notes/PHASE_1_FEATURES.md)

**References**
- Lindsey et al. (2025). [On the Biology of a Large Language Model](https://transformer-circuits.pub/2025/attribution-graphs/biology.html)
- Ameisen et al. (2025). [Circuit Tracing: Revealing Computational Graphs in Language Models](https://transformer-circuits.pub/2025/attribution-graphs/methods.html)
- [Neuronpedia](https://www.neuronpedia.org) · [`circuit-tracer`](https://github.com/safety-research/circuit-tracer) · [Gemma Scope](https://huggingface.co/google/gemma-scope-2b-pt-transcoders)

*Inspired by a reel from [@covacut](https://www.instagram.com/covacut/) on Instagram.*
