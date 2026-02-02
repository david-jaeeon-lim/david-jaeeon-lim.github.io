---
layout: page
title: "Phonetic-Aware Encoder Tuning"
description: "Boosting ASR Robustness against L2 Pronunciation Variations via CTC Supervision"
img: assets/img/projects/1_phonetic/phonetic_thumnail.png
importance: 1
category: 2025
related_publications: true
---

## Abstract

While large-scale pre-trained Automatic Speech Recognition (ASR) models like OpenAI's Whisper have achieved near-human performance on native speech, they often struggle with **non-native (L2) speech**. This is primarily due to **"Phonetic Disruption"**, where a speaker's native language (L1) interference leads to acoustic variations that deviate from standard pronunciation. In this project, we propose a **Phonetic-Aware Encoder Tuning** framework that leverages **LoRA (Low-Rank Adaptation)** and an auxiliary **CTC (Connectionist Temporal Classification)** head to enhance the encoder's acoustic resolution for L2 Korean speech.

---

## The Challenge: Phonetic Disruption in L2 Speech

L2 speakers often substitute target language phonemes with the most similar sounds from their native inventory. For instance:
* **Japanese (JP) speakers** may struggle with Korean nasal codas and stop/fricative distinctions.
* **Chinese (CN) speakers** often face challenges with specific vowel heights (e.g., [u] vs [o]).
* **Vietnamese (VN) speakers** frequently omit final consonants or mispronounce glottal fricatives.

To address these, our model must go beyond simple transcript mapping and learn the fine-grained acoustic-phonetic alignments of L2 speech.

---

## Proposed Methodology

### 1. Encoder-only LoRA Adaptation
Rather than fine-tuning the entire Whisper model—which could lead to catastrophic forgetting of its vast linguistic knowledge—we apply **LoRA** specifically to the **Encoder** blocks. This allows the model to adapt its feature extraction process to L2-specific acoustic patterns while keeping the pre-trained Decoder weights frozen.

### 2. Auxiliary CTC Supervision
We introduce an auxiliary **CTC Head** attached to the last layer of the Encoder. The objective function is a multi-task loss:

$$\mathcal{L}_{total} = \mathcal{L}_{Whisper\_Decoder} + \lambda \mathcal{L}_{CTC}$$

The CTC loss provides an explicit frame-level alignment signal, forcing the encoder to capture precise phonetic boundaries. This module is discarded during inference, ensuring no additional computational cost.

### 3. Comparing Phonetic Representations
We investigated which representation best guides the encoder:
* **Phonetic Hangul (g2pk2)**: Transcribing based on actual pronunciation rules.
* **Yale Romanization**: A morpho-phonemic representation of Korean.
* **International Phonetic Alphabet (IPA)**: Providing universal, fine-grained articulatory information.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/1_phonetic/model_architecture.png" title="Proposed Framework" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 1: Overview of the proposed Phonetic-Aware Encoder Tuning with CTC LoRA. (Use <b>Figure 1</b> from your report)
</div>

---

## Experimental Setup

* **Dataset**: AI Hub "Speech Data for Korean Language Learners" (Japanese, Chinese, and Vietnamese speakers).
* **Base Model**: Whisper-small.
* **Implementation**: HuggingFace Transformers with PEFT (LoRA).

---

## Results and Analysis

### Quantitative Performance
Our experiments show that **IPA-based supervision** consistently outperforms other methods across different L1 groups.

| Model | JP (CER↓) | CN (CER↓) | VN (CER↓) |
| :--- | :---: | :---: | :---: |
| Whisper Original | 0.0822 | 0.1171 | 0.1247 |
| **LoRA + IPA CTC** | **0.0768** | **0.1044** | **0.1149** |

### Phonetic Error Mitigation
By analyzing the confusion matrices, we observed that our method significantly reduces specific L1-induced errors:

* **Vowel Distinctions**: Improved accuracy in distinguishing [u] and [o] for Chinese speakers.
* **Coda Restoration**: Successful recognition of final nasals ([M], [N], [NG]) which are often confused by Japanese speakers.
* **Fricative Accuracy**: Enhanced detection of [H] and tense/lax consonant distinctions.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/projects/1_phonetic/error_matrix.png" title="Error Change Matrix" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Figure 2: Top 20 phonetic error reduction patterns. Negative values indicate a decrease in errors compared to the baseline. (Use <b>Figure 2</b> from your report)
</div>

---

## Conclusion

Our study demonstrates that providing **explicit phonetic guidance** through an auxiliary CTC task is a powerful strategy for adapting ASR models to non-native speech. By focusing the adaptation on the **Encoder** via LoRA, we successfully boosted robustness against L2 pronunciation variations without sacrificing the model's general language modeling capabilities.

Future work will involve exploring more diverse L1 groups and investigating the impact of noise-robust phonetic targets.

---