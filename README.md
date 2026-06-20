# Reviving Urdu Nastaliq

A comparative study of conventional computer vision, classical deep learning, and transformer plus LLM approaches for offline handwritten Urdu Nastaliq text recognition.

Live demo: https://huggingface.co/spaces/tamkeenSara/urdu-nastaliq-htr

Full paper: [report.pdf](report.pdf)

## Authors

This was a team project for the Computer Vision course at the School of Electrical Engineering and Computer Science (SEECS), National University of Sciences and Technology, Islamabad.

- Tamkeen Sara
- Fizza Kashif
- Muhammad Furqan Reza

This repository is an independent copy of the team's work, maintained for personal portfolio purposes. The canonical team repository is at [github.com/fizza49/Reviving-Urdu-Nastaliq](https://github.com/fizza49/Reviving-Urdu-Nastaliq).

## Motivation

Urdu Nastaliq is used by over 230 million speakers, yet offline handwritten text recognition for the script remains substantially behind Latin and East Asian scripts. The script's cascading diagonal baseline, fusion of characters into roughly 700 ligature classes, and variable diacritic positioning defeat most OCR systems designed for regular scripts. No prior published work had compared hand engineered features, classical deep learning, and transformer plus LLM approaches on identical datasets under identical writer independent conditions. This project closes that gap.

## What this project does

Three recognition pipelines are implemented and evaluated on a unified, writer independent corpus built from the NUST-UHWR and UNHD datasets:

1. **HOG + SVM** (conventional computer vision baseline). Histogram of Oriented Gradients features with projection profiles, classified with a linear SVM and a trigram language model for post processing. Operates at word level on a closed vocabulary.
2. **CNN-BiLSTM-CTC** (classical deep learning). A convolutional recurrent network with bidirectional LSTM layers and CTC decoding, trained with cosine annealing and GELU activations. Operates at line level with an open vocabulary.
3. **SwinHTR + mBART with LLM post-correction** (novel contribution). A Swin Transformer encoder paired with an mBART-50 decoder, fine tuned on Urdu, with a Groq hosted LLM used as a post correction stage over the decoder output.

Beyond the headline metrics, the project produces the first ligature class level error breakdown across all three pipelines, identifying which of the roughly 700 Nastaliq ligature classes drive recognition failure and why.

## Results

Evaluated on the writer independent held out test partition of NUST-UHWR:

| Pipeline | CER | WER |
| --- | --- | --- |
| HOG + SVM | 0.7908 | 0.9250 |
| CNN-BiLSTM-CTC (greedy) | 0.7688 | 1.0565 |
| CNN-BiLSTM-CTC (beam search) | 0.7669 | -- |
| SwinHTR + mBART (raw inference) | 0.7372 | 1.0040 |
| SwinHTR + mBART (fine tuned, best validation) | 0.7292 | 0.9877 |

The paradigm level ordering (transformer outperforms classical deep learning outperforms hand engineered features) holds even under the data constraints described below. Full ligature class breakdowns, bootstrap confidence intervals, and the LLM post correction ablation are in the [paper](report.pdf).

## Why the error rates look high

These CER values are elevated relative to published benchmarks such as ViLanOCR (1.1 percent CER) for a specific, well documented reason rather than an architectural shortcoming. The UOHTD dataset, a major source of diverse line level training samples in prior work, was inaccessible for this project. The dataset authors confirmed intent to grant access after a formal request submitted through ResearchGate, then stopped responding. Training was therefore restricted to NUST-UHWR and UNHD, which together cover only about 450 of the roughly 700 Nastaliq ligature classes with meaningful frequency. The paper discusses this constraint and its effect on results in detail under Limitations.

## Repository structure

```
.
├── urdu_nastaliq_htr.ipynb   Full experimental pipeline: data loading, preprocessing,
│                              all three models, training, evaluation, ligature analysis
├── report.pdf                 Full paper (PDF, rendered from the original Word report)
├── report.docx                 Original Word source of the paper
├── requirements.txt
└── .gitignore
```

## Reproducing this work

The notebook was developed and run on Kaggle, using a T4 GPU and Kaggle's dataset and secrets infrastructure. To reproduce it:

1. Add the two datasets through Kaggle's **Add Data** panel:
   - NUST-UHWR: `ameerhamzaqamar28/nust-uhwr-dataset`
   - UNHD: `drsaadbinahmed/unhd-dataset`
2. Add the following as Kaggle secrets if you want to run the LLM post correction stage and the Hugging Face Space push at the end of the notebook:
   - `GROQ_API_KEY` (and optionally `GROQ_API_KEY_1` through `GROQ_API_KEY_20` for parallel correction across multiple keys)
   - `HF_TOKEN` and `HF_REPO_ID`
3. Run the notebook top to bottom. Section 0 installs all dependencies; the same versions are pinned in `requirements.txt` for reference outside Kaggle.

Running outside Kaggle requires adjusting the `NUST_ROOT` and `UNHD_ROOT` paths near the top of Section 1 to point at your local copies of the datasets, and providing the same two secrets through environment variables or a `.env` file instead of `kaggle_secrets`.

## Datasets

- [NUST-UHWR](https://dll.seecs.nust.edu.pk/downloads/), NUST Urdu Handwritten Word Recognition Dataset, Document Layout and Language Laboratory, SEECS, NUST
- [UNHD](https://www.kaggle.com/datasets/drsaadbinahmed/unhd-dataset), Urdu Nastaliq Handwritten Dataset

## Limitations and future work

Evaluation is restricted to line level segmented text rather than full document page processing. The HOG and SVM baseline is closed vocabulary by construction. Full end to end LLM post correction on the complete test partition was constrained by API rate limits and is planned as an immediate extension. These points, along with five concrete directions for future work including targeted ligature data collection and ensemble approaches across pipelines, are detailed in the paper's Discussion and Future Work sections.

## Citation

If you build on this work, please cite the accompanying paper, included in this repository as `report.pdf`.
