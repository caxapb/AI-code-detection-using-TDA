# AI Code Detection Using Topological Data Analysis

This repository contains the experimental pipeline for detecting AI-generated source code using lexical features, code embeddings, and Topological Data Analysis (TDA). The project was developed as a reproducible research pipeline: it prepares benchmark data, audits possible leakage, extracts several feature families, trains logistic-regression detectors, and evaluates the models under ordinary and stress-test conditions.

## Project idea

The goal is to test whether topological descriptors can add useful structural information to AI-code detection. The pipeline compares three views of the same code fragment:

1. **Lexical and style features**: length, line statistics, comments, indentation, token counts, keyword ratios, and related surface-level code descriptors.
2. **Code embeddings**: pretrained vector representations such as CodeBERT, GraphCodeBERT, UniXcoder, SBERT, and Ada where available.
3. **Topological features**: persistence-based descriptors computed from code-derived point clouds, including compact persistence summaries and extended Betti-curve features.

TDA is not treated as a replacement for strong code embeddings. It is evaluated as a structural component that can help diagnose surface dependence, stress-test robustness, and sometimes improve selected configurations.

## Datasets and cached embeddings

All the notebooks were run on the Kaggle platform. The data was loaded there manually and produced generated embeddings during the execution are also loaded back to Kaggle as a separate Dataset.

Kaggle dataset with original row data: [link](https://www.kaggle.com/datasets/caxapb/tda-data/data)

Kaggle dataset with cached embeddings: [link](https://www.kaggle.com/datasets/caxapb/all-tda-embs)

The dataset-output of the Oedingen notebook run: [link](https://www.kaggle.com/datasets/caxapb/tda-embs-human-like)

For Oedingen, Ada syntax-normalized embeddings are not used because they cannot be regenerated without access to the same external embedding service.

## Benchmarks

### Oedingen

The Oedingen benchmark is used as the controlled Python setting. The preprocessing stage performs duplicate and leakage control with several auxiliary signatures:

- exact code signature;
- whitespace-normalized signature;
- commentless signature;
- AST-normalized signature.

These normalized views are used for duplicate auditing and grouping only. The detector is trained on the original code representation and on features extracted from the saved sample.

### CodeMirage

CodeMirage is used as the external multilingual benchmark. It contains human-written and AI-generated code across multiple programming languages, generators, and paraphrased variants. Since Ada embeddings are not available for CodeMirage, the CodeMirage experiments rely on open-source embedding families and cached HuggingFace embeddings.

## Model design

The final experimental setup uses **Logistic Regression**.

The current design asks one main question: what changes when different feature groups are given to the same simple classifier?

The evaluated configurations include:

- lexical features only;
- TDA features only;
- lexical + TDA;
- embedding only;
- embedding + lexical;
- embedding + TDA;
- embedding + lexical + TDA.

This design keeps the classifier fixed and makes the feature contribution easier to interpret.

## Topological feature extraction

For the topology-aware part of the pipeline, code is converted into a point cloud and persistent homology is computed on that representation.

For Oedingen, Python code can be represented through syntax-tree-derived local structures. For CodeMirage, a more general token-level representation is used because the dataset contains several programming languages.

The TDA features include:

- counts of persistent structures;
- total persistence;
- maximum and mean persistence;
- persistence entropy;
- Betti-curve coordinates.

In the result tables, the compact TDA feature block is usually referred to as `TDA-summary`, while the extended block with Betti curves is referred to as `TDA-all`.

## Evaluation

The notebooks export tables and figures for several evaluation settings:

- fixed validation and test splits;
- controlled feature-addition comparisons;
- feature-volume analysis;
- repeated splits;
- source-shift or generator-shift checks;
- paraphrase stress testing for CodeMirage;
- surface-normalization stress testing;
- calibration and error analysis.

The main metrics are F1, ROC-AUC, PR-AUC, MCC, Brier score, and McNemar's test for paired error comparison. F1 is used as the main classification score, ROC-AUC and PR-AUC describe ranking quality, MCC summarizes the full confusion matrix, Brier score evaluates probability calibration, and McNemar's test checks whether adding a feature block changes the error set in a meaningful paired comparison.

## Syntax-normalization stress test

The surface-normalization stress test is separate from duplicate auditing. During duplicate auditing, normalized code variants are used only to compute signatures and group similar snippets. During the stress test, the test code itself is transformed: comments and docstrings are removed, whitespace is normalized, and identifiers are replaced with generic names.

For this test, lexical features, TDA features, and available HuggingFace embeddings are recomputed or loaded from cached syntax-normalized matrices. The model is still trained on the original training representation, so this experiment measures inference-time robustness to surface editing.

## Repository workflow

A typical run follows this order:

1. Prepare or attach the required Kaggle datasets.
2. Run the Oedingen notebook.
3. Run the CodeMirage notebook.
4. Check exported CSV tables and PNG figures in the output folders.
5. Use the generated artifacts in the thesis text or further analysis.

The notebooks are written to prefer cached embeddings. If cached embeddings are present in the attached Kaggle dataset, they are loaded from disk. Expensive embedding generation should be avoided unless explicitly needed.


## Key findings

Across the latest experiments, pretrained embeddings remain the strongest primary signal. On Oedingen, Ada-based configurations dominate because Ada embeddings are available for that dataset. On CodeMirage, where Ada is not available, GraphCodeBERT is the strongest fixed-test representation.

TDA has a more selective role. It can improve some configurations and stress-test behavior, but it does not dominate every benchmark. Its main value is structural and diagnostic: it helps show how much the detector depends on surface-level style, how errors change when topology is added, and how feature families behave under distribution shifts.

## Requirements

The notebooks are intended to run on Kaggle. The exact environment may depend on the notebook version, but the main Python packages are enumerated in the requirements.txt

GPU is useful for embedding generation. If all cached embeddings are already available, most evaluation steps can run without regenerating embeddings.

## Notes on reproducibility

- Keep dataset versions fixed when comparing results.
- Use the same saved embeddings and split settings when reproducing tables.
- For stress tests with embeddings, use syntax-normalized embeddings that match the same dataset order and split.

