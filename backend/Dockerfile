FROM python:3.13.15-slim

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir --upgrade pip

RUN pip install --no-cache-dir \
    torch \
    --index-url https://download.pytorch.org/whl/cpu

RUN pip install --no-cache-dir -r requirements.txt

ENV HF_HOME=/app/huggingface

# Download the model at build time
RUN python -c "\
from langchain_huggingface import HuggingFaceEmbeddings; \
HuggingFaceEmbeddings( \
    model_name='BAAI/bge-base-en-v1.5', \
    model_kwargs={'device': 'cpu'}, \
    encode_kwargs={'normalize_embeddings': True} \
)"

# Force offline mode at runtime so it uses the cached model
ENV TRANSFORMERS_OFFLINE=1

ENV HF_HUB_OFFLINE=1

# Starts 1 worker process to handle requests.
# A worker can take up to 120 seconds without reporting activity before Gunicorn terminates and restarts it.
# Listens on port 7860 across all network interfaces inside the container.
# Imports the variable rag_api from the Python module app
CMD ["gunicorn", "-w", "1", "--timeout", "240", "-b", "0.0.0.0:7860", "app:rag_api"]
