# Semantic Search with FAISS

A semantic search engine built with **Hugging Face Datasets**, **Transformers**, and **FAISS** to search GitHub issue comments by meaning instead of exact keyword matching.

## Overview

Traditional search usually depends on exact keyword matches. This project takes a different approach by using **semantic search**, where both documents and user queries are converted into dense vector representations called **embeddings**. These embeddings are then indexed with **FAISS** for efficient nearest-neighbor search.

The search engine is built on a dataset of GitHub issues and comments from the Datasets repository. Each issue comment is enriched with its related issue title and body, embedded into a vector space, and indexed for fast similarity search.

## What this project does

- Loads GitHub issues from the Hugging Face dataset
- Removes pull requests and low-value entries
- Explodes issue comments so each row contains one comment
- Builds a combined text field from:
  - issue title
  - issue body
  - issue comment
- Generates embeddings using a Transformer model
- Indexes embeddings using **FAISS**
- Searches the corpus using natural language questions
- Returns the most semantically similar issue comments

## Why semantic search?

Keyword-based search can miss relevant results when the query uses different wording than the document.

Semantic search helps solve this by finding results based on **meaning**, not just matching words.

For example:

- Query: `How can I load a dataset offline?`
- A relevant result may mention:
  - cached datasets
  - local loading
  - internet connection issues
  - offline mode

Even if the exact words do not match, semantic search can still retrieve useful results.

## Dataset

This project uses the following dataset:

- `lewtun/github-issues`

The dataset contains GitHub issues and comments. For this project, only useful issue discussions are kept.

### Filtering steps

The dataset is cleaned in several stages:

1. Remove pull requests
2. Remove issues with no comments
3. Keep only the important columns:
   - `title`
   - `body`
   - `html_url`
   - `comments`
4. Explode the `comments` column so each row contains a single comment
5. Remove very short comments
6. Concatenate title, body, and comment into one searchable text field

## Model used

This project uses the following embedding model:

- `sentence-transformers/multi-qa-mpnet-base-dot-v1`

This model is well-suited for **asymmetric semantic search**, where:

- the query is short
- the target documents are longer

## Tech stack

- Python
- PyTorch
- Hugging Face Transformers
- Hugging Face Datasets
- FAISS
- Pandas

## Project workflow

### 1. Load dataset

The GitHub issues dataset is loaded using `load_dataset()`.

### 2. Clean and prepare data

The project filters out:

- pull requests
- issues without comments
- short comments with little search value

### 3. Expand comments

Since each issue may contain multiple comments, the `comments` column is exploded so that each row becomes:

- one issue title
- one issue body
- one issue comment
- one source URL

### 4. Build searchable text

A new `text` field is created by joining:

- title
- body
- comment

This gives better context for embeddings.

### 5. Create embeddings

Each text is converted into a dense vector using the Transformer model.

The project applies **CLS pooling** to obtain a single vector representation for each text.

### 6. Build FAISS index

The embeddings are indexed with FAISS to enable fast similarity search.

### 7. Search with natural language

A user question is also embedded and compared with all indexed comment embeddings. The top matching results are returned.

## Example query

```python
question = "How can I load a dataset offline?"
