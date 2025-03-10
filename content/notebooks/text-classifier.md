---
title: "Text Classifier"
url: "/ldd/notebooks/text-classifier/"
---

```python
import pandas as pd
import numpy as np
import torch
from datasets import Dataset, load_from_disk
from transformers import AutoTokenizer, AutoModel
import faiss
from transformers import PegasusTokenizer, PegasusForConditionalGeneration

```

    /home/matias/anaconda3/envs/new_env/lib/python3.11/site-packages/tqdm/auto.py:21: TqdmWarning: IProgress not found. Please update jupyter and ipywidgets. See https://ipywidgets.readthedocs.io/en/stable/user_install.html
      from .autonotebook import tqdm as notebook_tqdm



```python
import torch
from transformers import pipeline, AutoModelForSequenceClassification, AutoTokenizer

# --- Step 1: Load Model & Tokenizer ---
model_name = "microsoft/deberta-large-mnli"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

# Move model to GPU if available
device = "cuda" if torch.cuda.is_available() else "cpu"
model.to(device)

# --- Step 2: Classify Text ---
classifier = pipeline("zero-shot-classification", model=model, tokenizer=tokenizer, device=0 if torch.cuda.is_available() else -1)

text = "This book describes the history of artificial intelligence and its impact on society."
labels = ["technology", "history", "science", "fiction"]

result = classifier(text, candidate_labels=labels, multi_label=True)

print(result)

```

    Some weights of the model checkpoint at microsoft/deberta-large-mnli were not used when initializing DebertaForSequenceClassification: ['config']
    - This IS expected if you are initializing DebertaForSequenceClassification from the checkpoint of a model trained on another task or with another architecture (e.g. initializing a BertForSequenceClassification model from a BertForPreTraining model).
    - This IS NOT expected if you are initializing DebertaForSequenceClassification from the checkpoint of a model that you expect to be exactly identical (initializing a BertForSequenceClassification model from a BertForSequenceClassification model).
    Device set to use cpu
    Asking to truncate to max_length but no maximum length is provided and the model has no predefined maximum length. Default to no truncation.


    {'sequence': 'This book describes the history of artificial intelligence and its impact on society.', 'labels': ['technology', 'history', 'science', 'fiction'], 'scores': [0.9525855779647827, 0.9177680015563965, 0.33374109864234924, 0.002266060095280409]}

