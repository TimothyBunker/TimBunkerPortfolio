---
title: Character-Level Tagging with Multi-Dimensional Positional Transformers
description: Enhancing character-based sequence tagging (POS, NER) by injecting multiple positional signals (absolute, within-word, word-index) into a Transformer model.
date: 2025-04-01 22:13:00 
label: NLP # Or Deep Learning, Research, Project
image: '/TimBunkerPortfolio/images/multi-dimensional-transformer.png' 
---

# Character-Level Tagging with Multi-Dimensional Positional Transformers

## Project Overview

This project explores an enhanced Transformer architecture for character-level sequence tagging tasks like Part-of-Speech (POS) tagging or Named Entity Recognition (NER). The core innovation lies in augmenting standard character embeddings with multiple, explicit positional signals before feeding them into a Transformer encoder. These signals include the character's absolute position in the sequence, its position *within* its word, and the index of the word it belongs to.

The goal is to provide the model with richer structural context directly at the character level, potentially improving its ability to handle morphology, out-of-vocabulary words, and fine-grained tagging decisions compared to purely word-based or simpler character-based models.

---

## Purpose

### Why Character-Level Tagging with Enhanced Positional Info?
Traditional NLP models often operate on pre-tokenized words, which can struggle with unknown words (OOV problem) and lose potentially valuable morphological information encoded within words. Character-level models naturally handle these issues but can sometimes lack a clear sense of word boundaries and larger sentence structure.

This project hypothesizes that explicitly providing multiple "dimensions" of positional information can bridge this gap:
- **Leverage Character Benefits:** Retain the ability to handle morphology and OOV words.
- **Inject Word Structure:** Explicitly inform the model about word boundaries and internal word structure via `within-word` and `word-index` positions.
- **Utilize Transformer Power:** Employ the proven strength of Transformer encoders for capturing long-range dependencies in the sequence context.

This approach aims to create a more informed character-level representation for robust sequence tagging.

---

## Key Features

### 1. **Multi-Dimensional Positional Inputs**
The model integrates several distinct positional signals:
- **Character Embeddings**: Standard embeddings for each character in the vocabulary.
- **Absolute Positional Encoding**: Sinusoidal encoding capturing the character's overall sequence position.
- **Within-Word Position Embedding**: A learned embedding based on the character's index *within* its parent word (e.g., 0 for 'T' in "The", 1 for 'h', 2 for 'e').
- **Word Index Embedding**: A learned embedding based on the index of the word the character belongs to in the sentence (e.g., 0 for all chars in "The", 1 for all chars in "quick", etc.).

### 2. **Embedding Combination Strategy**
Character, within-word position, and word index embeddings are summed element-wise. Absolute positional encoding is subsequently added, creating a rich initial representation for each character token before the main Transformer layers.

### 3. **Standard Transformer Encoder Backbone**
The core sequence processing relies on a stack of standard `nn.TransformerEncoderLayer` modules, utilizing multi-head self-attention to model contextual dependencies across the character sequence.

### 4. **Automated Data Acquisition Pipeline (`download_ud.py`)**
Includes a utility script to automatically download and preprocess Universal Dependencies (UD) treebanks:
- Clones/pulls specified UD repositories from GitHub.
- Parses `.conllu` files to extract raw sentences.
- Organizes data into `train.txt`, `valid.txt`, and `test.txt` (using a designated primary treebank for validation/test sets), ready for further preprocessing into model inputs.

---

## Design Considerations and Approach

While not "challenges" in the sense of debugging roadblocks, key design choices were made to address potential limitations of character-level modeling:

### 1. **Explicitly Encoding Structure**
- **Problem**: Pure character models might struggle to infer word boundaries robustly.
- **Approach**: Directly injecting `within-word` and `word-index` embeddings provides this structural information explicitly, reducing the learning burden on the self-attention mechanism.

### 2. **Leveraging Existing Architectures**
- **Problem**: Developing entirely novel sequence models is complex.
- **Approach**: Built upon the well-established and powerful Transformer encoder architecture, focusing innovation on the input representation layer.

### 3. **Handling Data Complexity**
- **Problem**: Generating the required multi-dimensional positional indices requires careful preprocessing.
- **Approach**: The architecture assumes a preprocessing pipeline capable of accurately mapping each character to its character index, within-word position, word index, and target tag. The `download_ud.py` script provides the *initial* sentence extraction step.

---

## Results (Implementation Status)

### Current Status
- The core `MultiLevelPosTransformer` model architecture is implemented in PyTorch (`model.py`).
- A data acquisition script (`download_ud.py`) is available to fetch and prepare raw sentence data from UD treebanks.

### Expected Outcomes
- The model is hypothesized to perform competitively or potentially outperform baseline character-level models that lack explicit word-level positional signals, especially on tasks sensitive to morphology or involving OOV words.
- Evaluation on standard POS tagging or NER benchmarks is the next step to validate performance.

*(Note: Update this section with concrete results once experiments are conducted)*

---

## Future Work

### 1. **Preprocessing Implementation**
- Develop robust scripts to convert raw sentences (from `train/valid/test.txt`) into the required tensor formats: `chars`, `within_word_pos`, `word_indices`, `target_tags`, and `pad_mask`.
- Implement PyTorch `Dataset` and `DataLoader` classes.

### 2. **Training and Evaluation**
- Implement a training script (`train.py`) incorporating the model, data loaders, loss function (CrossEntropyLoss with padding ignore), and optimizer (e.g., AdamW).
- Conduct systematic evaluations on benchmark datasets (e.g., UD English EWT for POS tagging).

### 3. **Ablation Studies**
- Train and evaluate model variants by removing specific positional embeddings (e.g., only absolute + char, only absolute + char + within-word) to quantify the contribution of each component.

### 4. **Hyperparameter Tuning**
- Optimize `d_model`, `nhead`, `num_encoder_layers`, learning rate, etc.

### 5. **Cross-Lingual Application**
- Test the model's effectiveness on morphologically richer languages where character-level information is potentially even more crucial.

---

## Conclusion

This project presents a novel approach to character-level sequence tagging by enriching input representations with multiple, explicit positional dimensions. By combining the flexibility of character-level processing with structural information about words and their positions, the `MultiLevelPosTransformer` aims to provide a robust and potentially more accurate alternative for tasks like POS tagging and NER. The foundational model and data acquisition tools are implemented, paving the way for empirical evaluation and further development.

---

## Repository and Resources

### Code
The primary model implementation and data download script can be found [here](https://github.com/TimothyBunker/MultiDimensionalTransformer). 
Key files:
- `model.py`: Defines the `MultiLevelPosTransformer` and `PositionalEncoding` classes.
- `download_ud.py`: Script to download and prepare raw UD sentence data.
- **Note:** Preprocessing, training, and evaluation scripts need to be implemented separately based on the requirements outlined above.

### Dependencies
- Python 3.x
- PyTorch (>= 1.8 recommended)
- Git (for `download_ud.py`)

Refer to the repository's `README.md` for detailed setup and potential future updates on usage instructions.

---
