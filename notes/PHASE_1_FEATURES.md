# Phase 1: Feature Labels (English / French antonym circuit)

**Model:** Gemma-2-2B · **Transcoders:** Gemma Scope per-layer (16k)
**Notebook:** [`01_trace_en_fr.ipynb`](../notebooks/01_trace_en_fr.ipynb)

| Language | Prompt | Top-5 predictions |
|---|---|---|
| English | `The opposite of "small" is "` | |
| French | `Le contraire de "petit" est "` | |

## Graph scores

| Language | Replacement score | Completeness score |
|---|---|---|
| English | | |
| French | | |

## Shared features (middle layers, final token)

Candidates for the language-independent **antonym** and **large/big** concepts.

| Layer | Feature # | Label | Evidence (top activating examples) | Neuronpedia link |
|---|---|---|---|---|
| | | | | |

## Language-specific features

### Early layers (input side)

| Layer | Feature # | Language | Label | Neuronpedia link |
|---|---|---|---|---|
| | | | | |

### Late layers (output side)

| Layer | Feature # | Language | Label (e.g. say *grand*) | Neuronpedia link |
|---|---|---|---|---|
| | | | | |

## Operand features (`small` / `petit` token)

| Layer | Feature # | Label | Shared? | Neuronpedia link |
|---|---|---|---|---|
| | | | | |

## Overlap plot

![Feature overlap EN vs FR](../figures/01_overlap_en_fr.png)

- Does EN–FR overlap beat both baselines? In which layers?
-

## Intervention results

| Features suppressed | EN top-5 before → after | FR top-5 before → after |
|---|---|---|
| | | |

## Conclusion

- **Shared concept space replicated?** Yes / Partly / No
- **Notes:**
