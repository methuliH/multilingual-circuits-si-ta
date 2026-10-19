# Multilingual Circuits in Sinhala and Tamil

**Do language models use a shared, language-independent concept space for low-resource languages with their own scripts?**

> 🚧 **Work in progress.** Results will be added as the experiments complete.

## Motivation

Anthropic's [*On the Biology of a Large Language Model*](https://transformer-circuits.pub/2025/attribution-graphs/biology.html) found that Claude 3.5 Haiku processes the same concept in English, French and Chinese using **shared, language-independent features** in its middle layers, with language-specific features only at the input and output. They also found that:

- concept sharing **increases with model size**, and
- the gain is **largest for languages that don't share an alphabet**.

That points to a hard case nobody has tested: **a small model on low-resource languages with their own scripts.** This project replicates the multilingual-circuits finding on **Gemma-2-2B** and extends it to **Sinhala (සිංහල)** and **Tamil (தமிழ்)**.

## Research questions

1. Does Gemma-2-2B reuse the same concept features for Sinhala and Tamil as for English and French?
2. Does feature overlap peak in the middle layers for Sinhala and Tamil, as it does for other languages?
3. Do interventions using features found in **English** prompts (swapping the operation, operand or output language) also work in Sinhala and Tamil?
4. Is English a stronger "default" language for Sinhala and Tamil than for French?

## Method

- **Model:** Gemma-2-2B
- **Tools:** [`circuit-tracer`](https://github.com/safety-research/circuit-tracer) for attribution graphs and interventions; [Neuronpedia](https://www.neuronpedia.org/gemma-2-2b/graph) for interactive graphs
- **Task:** antonym completion, following the original paper, e.g. `The opposite of "small" is "` → *big*
- **Analysis:**
  - Attribution graphs for each prompt × language
  - Feature overlap (intersection-over-union) between translated prompts, per layer, against an unrelated-prompt baseline
  - Feature interventions (suppression and injection) to test each claimed mechanism
  - Replacement and completeness scores compared across languages, to check how well the transcoders cover each script

## Repository structure

```
.
├── notebooks/   # Colab-ready experiment notebooks
├── data/        # Prompt sets for each language
├── figures/     # Graphs and plots
└── notes/       # Paper reading notes and findings
```

## Running the notebooks

The notebooks are designed for **Google Colab** with a free T4 GPU.

1. Accept the [Gemma-2-2B license](https://huggingface.co/google/gemma-2-2b) on Hugging Face
2. Create a Hugging Face **read** token
3. In Colab, add it under **🔑 Secrets** as `HF_TOKEN` and enable notebook access
4. **Runtime → Change runtime type → T4 GPU**, then **Run all**

## Results

*Coming soon.*

## References

- Lindsey et al. (2025). [On the Biology of a Large Language Model](https://transformer-circuits.pub/2025/attribution-graphs/biology.html). Transformer Circuits Thread.
- Ameisen et al. (2025). [Circuit Tracing: Revealing Computational Graphs in Language Models](https://transformer-circuits.pub/2025/attribution-graphs/methods.html). Transformer Circuits Thread.
- [Open-sourcing circuit-tracing tools](https://anthropic.com/research/open-source-circuit-tracing), Anthropic
- [`safety-research/circuit-tracer`](https://github.com/safety-research/circuit-tracer)

## Acknowledgements

This project builds on the circuit-tracing methods and tools released by Anthropic and the `circuit-tracer` authors, the Gemma Scope transcoders from Google DeepMind, and Neuronpedia.
