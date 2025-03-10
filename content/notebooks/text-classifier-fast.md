---
title: "Text Classifier Fast"
url: "/ldd/notebooks/text-classifier-fast/"
---

```python
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.model_selection import train_test_split

# Load dataset with 20 broad categories
categories = [
    "sci.space", "sci.electronics", "sci.med", "comp.graphics", "comp.os.ms-windows.misc",
    "rec.sport.baseball", "rec.sport.hockey", "talk.politics.misc", "talk.religion.misc",
    "misc.forsale"
]
newsgroups = fetch_20newsgroups(subset="all", categories=categories, remove=("headers", "footers", "quotes"))

# Convert text to TF-IDF feature vectors
vectorizer = TfidfVectorizer(stop_words="english", max_features=5000)  # Limit feature size for speed
X = vectorizer.fit_transform(newsgroups.data)
y = newsgroups.target  # Category labels

# Split into train and test sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

```


```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score

# Train the Logistic Regression classifier
logreg = LogisticRegression(max_iter=1000)
logreg.fit(X_train, y_train)

# Evaluate on test set
y_pred = logreg.predict(X_test)
print("Logistic Regression Accuracy:", accuracy_score(y_test, y_pred))

```

    Logistic Regression Accuracy: 0.8073196986006459



```python
from sklearn.naive_bayes import MultinomialNB

# Train the Naïve Bayes classifier
nb = MultinomialNB()
nb.fit(X_train, y_train)

# Evaluate on test set
y_pred_nb = nb.predict(X_test)
print("Naïve Bayes Accuracy:", accuracy_score(y_test, y_pred_nb))

```

    Naïve Bayes Accuracy: 0.8089343379978472



```python
def classify_text(text, model):
    X_new = vectorizer.transform([text])  # Convert text to numerical features
    predicted_category = newsgroups.target_names[model.predict(X_new)[0]]
    return predicted_category

sample_text = "The recent discovery of a new exoplanet has excited many astronomers."
print("Predicted Category (Logistic Regression):", classify_text(sample_text, logreg))
print("Predicted Category (Naïve Bayes):", classify_text(sample_text, nb))

```

    Predicted Category (Logistic Regression): sci.space
    Predicted Category (Naïve Bayes): sci.space



```python
import os
import re

# Cargar modelo de spaCy en español y stopwords
import spacy
nlp = spacy.load("es_core_news_sm")

# import nltk
from nltk.corpus import stopwords
spanish_stopwords = set(stopwords.words("spanish"))

# Precompilar la regex para limpiar el texto
CLEAN_PATTERN = re.compile(r"<.*?>|[^a-zA-ZáéíóúüñÁÉÍÓÚÜÑ\s]")
CHUNKS_DIR = '/home/matias/RAG_master/storage/chunks'

def clean_text_batch(texts: list[str]) -> list[str]:
    """
    Limpia y lematiza un lote de textos utilizando spaCy en modo pipe.
    """
    cleaned_texts = []
    # Preprocesar cada texto con regex
    preprocessed_texts = [CLEAN_PATTERN.sub(" ", text).lower().strip() for text in texts]
    
    # Procesar el lote con nlp.pipe (esto es más rápido)
    for doc in nlp.pipe(preprocessed_texts, batch_size=32):
        # Filtrar stopwords y obtener lemas en un solo paso
        tokens = [token.lemma_ for token in doc if token.text not in spanish_stopwords]
        cleaned_texts.append(" ".join(tokens))
    return cleaned_texts

# %%
# Cargar textos de chunks en una lista (en vez de procesarlos uno por uno)
def load_all_chunk_texts(chunk_ids: list[str]) -> list[tuple[str, str]]:

    """
    Carga todos los textos de chunks dados sus IDs. 
    Se asume que los textos se encuentran en './chunks/<chunk_id>.txt'.
    Retorna una lista de tuplas (chunk_id, raw_text).
    """

    texts = []
    for chunk_id in chunk_ids:
        file_path = os.path.join(CHUNKS_DIR, f"{chunk_id}.txt")
        if os.path.isfile(file_path):
            with open(file_path, "r", encoding="utf-8") as f:
                raw_text = f.read()
                if raw_text.strip():
                    texts.append((chunk_id, raw_text))
    return texts

```


```python
import pandas as pd
from chunk_query import ChunkQuery  # Tu clase de consulta, tal como la definimos antes

CHUNKS_METADATA_FILE= "/home/matias/RAG_master/storage/tables/chunks_index.json"

df_metadata = pd.read_json(CHUNKS_METADATA_FILE, orient="index")

# 2. Inicializar la clase de consulta y filtrar los chunks de la carpeta /Academic/
cq = ChunkQuery(df_metadata)


# Ejemplo de query: Filtrar aquellos chunks cuyo 'original_path' contenga '/Academic/'
academic_chunk_ids = cq.query_custom("original_path.str.contains('/Academic/1_Input_Raw/Drafts_Papers_Early_Drafts/2024_1/AGGVOL')")
print("Cantidad de chunks en /Academic/1_Input_Raw/Drafts_Papers_Early_Drafts:", len(academic_chunk_ids))

# Cargar todos los textos
all_chunks = load_all_chunk_texts(academic_chunk_ids[:100])  # academic_chunk_ids es la lista filtrada
chunk_ids, raw_texts = zip(*all_chunks)  # Separar IDs y textos
chunk_ids, cleaned_texts = chunk_ids, clean_text_batch(raw_texts[:100])  # Separar IDs y textos

```


    ---------------------------------------------------------------------------

    FileNotFoundError                         Traceback (most recent call last)

    Cell In[6], line 6
          2 from chunk_query import ChunkQuery  # Tu clase de consulta, tal como la definimos antes
          4 CHUNKS_METADATA_FILE= "/home/matias/RAG_master/storage/tables/chunks_index.json"
    ----> 6 df_metadata = pd.read_json(CHUNKS_METADATA_FILE, orient="index")
          8 # 2. Inicializar la clase de consulta y filtrar los chunks de la carpeta /Academic/
          9 cq = ChunkQuery(df_metadata)


    File ~/anaconda3/envs/new_env/lib/python3.11/site-packages/pandas/io/json/_json.py:791, in read_json(path_or_buf, orient, typ, dtype, convert_axes, convert_dates, keep_default_dates, precise_float, date_unit, encoding, encoding_errors, lines, chunksize, compression, nrows, storage_options, dtype_backend, engine)
        788 if convert_axes is None and orient != "table":
        789     convert_axes = True
    --> 791 json_reader = JsonReader(
        792     path_or_buf,
        793     orient=orient,
        794     typ=typ,
        795     dtype=dtype,
        796     convert_axes=convert_axes,
        797     convert_dates=convert_dates,
        798     keep_default_dates=keep_default_dates,
        799     precise_float=precise_float,
        800     date_unit=date_unit,
        801     encoding=encoding,
        802     lines=lines,
        803     chunksize=chunksize,
        804     compression=compression,
        805     nrows=nrows,
        806     storage_options=storage_options,
        807     encoding_errors=encoding_errors,
        808     dtype_backend=dtype_backend,
        809     engine=engine,
        810 )
        812 if chunksize:
        813     return json_reader


    File ~/anaconda3/envs/new_env/lib/python3.11/site-packages/pandas/io/json/_json.py:904, in JsonReader.__init__(self, filepath_or_buffer, orient, typ, dtype, convert_axes, convert_dates, keep_default_dates, precise_float, date_unit, encoding, lines, chunksize, compression, nrows, storage_options, encoding_errors, dtype_backend, engine)
        902     self.data = filepath_or_buffer
        903 elif self.engine == "ujson":
    --> 904     data = self._get_data_from_filepath(filepath_or_buffer)
        905     self.data = self._preprocess_data(data)


    File ~/anaconda3/envs/new_env/lib/python3.11/site-packages/pandas/io/json/_json.py:960, in JsonReader._get_data_from_filepath(self, filepath_or_buffer)
        952     filepath_or_buffer = self.handles.handle
        953 elif (
        954     isinstance(filepath_or_buffer, str)
        955     and filepath_or_buffer.lower().endswith(
       (...)
        958     and not file_exists(filepath_or_buffer)
        959 ):
    --> 960     raise FileNotFoundError(f"File {filepath_or_buffer} does not exist")
        961 else:
        962     warnings.warn(
        963         "Passing literal json to 'read_json' is deprecated and "
        964         "will be removed in a future version. To read from a "
       (...)
        967         stacklevel=find_stack_level(),
        968     )


    FileNotFoundError: File /home/matias/RAG_master/storage/tables/chunks_index.json does not exist



```python

```
