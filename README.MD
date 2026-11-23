# Latin → English Translation with LoRA

Fine-tuning a Latin language model to translate Latin into English, using QLoRA (4-bit quantization + LoRA), as the final project for [course name].

> **Status:** work in progress. The notebooks have not yet been run end-to-end; results below are placeholders.

## Overview
| | |
|---|---|
| **Task** | Latin → English sentence translation |
| **Base model** | [`hathibelagal/llama-3.2-latin`](https://huggingface.co/hathibelagal/llama-3.2-latin) (3B, decoder-only, fine-tuned from Llama 3.2 3B) |
| **Method** | QLoRA: 4-bit NF4 base model + LoRA adapters (`peft`, `bitsandbytes`) |
| **Public data** | [`grosenthal/latin_english_parallel`](https://huggingface.co/datasets/grosenthal/latin_english_parallel) (~101k pairs, MIT) |
| **Our new data** | Hand-aligned pairs from the [Perseus Digital Library](https://www.perseus.tufts.edu) (`data/own_pairs.csv`, [N] pairs) |
| **Metrics** | chrF, BLEU, perplexity, manual error review |

Translation is framed as text generation. Each training example is:

```
Latin: <Latin sentence>
English: <English translation>
```

The loss is computed on the English part only.

## Pipeline
1. **Collect:** load the public corpus; add our own aligned pairs (`file` column = source work).
2. **Clean:** Unicode normalization, remove markup and footnote markers, drop empty rows and misaligned pairs (extreme length ratio), remove duplicates. We do **not** lowercase, remove stop words, stem or lemmatize: Latin endings carry grammatical information that translation needs.
3. **Split:** by source work (not by row) to avoid near-duplicate leakage; remove training sentences that also appear in validation/test; save the fixed splits.
4. **Tokenize:** the base model's own tokenizer; prompt tokens masked from the loss.
5. **Baseline:** evaluate the untuned model on the fixed test set.
6. **Fine-tune:** QLoRA (`r=16`, `lora_alpha=32`, attention projections, 1 epoch to start).
7. **Evaluate:** same test set; scores per domain (Vulgate vs. other); manual review of ~20 outputs.
8. **Save:** only the LoRA adapter.

## Repository structure
```
.
├── README.md
├── notebooks/
│   ├── latin_translation_lora.ipynb   # minimal pipeline, every line commented
├── data/
│   └── own_pairs.csv                                   # our hand-aligned pairs: la,en,file
└── results/
    └── (scores, sample outputs)
```

1. Open a notebook in Colab (GPU runtime) or locally.
2. Start with the `simple_commented` notebook (`n_train = 2000`) to check that everything runs.
3. Use the `stepbystep` notebook for the real experiments. Settings are in the `CONFIG` dictionary. When running locally, remove the Google Drive mount lines and change `save_dir`.

## Results
| Model | Perplexity | chrF | BLEU |
|---|---|---|---|
| Base (no fine-tuning) | – | – | – |
| + LoRA fine-tuning | – | – | – |


## Attribution and references
- Notebook structure adapted from the Week 1 tutorial *movie_review_sentiment_classifier_with_bert.ipynb*; task, model, data handling and metrics were changed for translation.
- Perseus Digital Library, Tufts University. Cite individual works and translators as required by each text's license.
