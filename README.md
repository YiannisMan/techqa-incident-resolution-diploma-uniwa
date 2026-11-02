# Automated IT Incident Resolution with Hybrid RAG & Fine-Tuned LLMs (ENGLISH)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)
![Llama3](https://img.shields.io/badge/LLM-Llama--3%20(QLoRA)-orange)
![ChromaDB](https://img.shields.io/badge/VectorDB-Chroma-green)
![Jira](https://img.shields.io/badge/Integration-Jira%20API-0052CC)

> **Diploma Thesis** | Department of Informatics and Computer Engineering, University of West Attica (UniWA)

## Overview

This repository contains an end-to-end AI-powered system designed for **automated IT incident resolution and technical support** using the **IBM TechQA** dataset. The system combines **Hybrid Retrieval-Augmented Generation (RAG)**, a custom **fine-tuned Cross-Encoder Reranker**, and a **QLoRA fine-tuned Llama 3** model, seamlessly integrated with **Jira** to automatically process, answer, and report on technical support tickets.

---

## Key Features & Architecture

1. **Hybrid Retrieval Pipeline (`app/retrieval/`)**:
   * **Sparse Retrieval**: Lexical search powered by **BM25** (`build-bm25-techqa.ipynb`).
   * **Dense Retrieval**: Semantic vector search using **ChromaDB** (`chroma_data/`, `ingest.py`).
2. **Custom Fine-Tuned Reranker (`app/reranker_model/`)**:
   * Fine-tuned cross-encoder model (`techqa-reranker-v2-finetune.ipynb`) that re-scores retrieved documents to maximize context relevance before generation.
3. **LLM Fine-Tuning & Deployment (`finetuning/`)**:
   * Supervised Fine-Tuning (SFT) dataset construction (`build_sft_dataset.py`).
   * **QLoRA Fine-Tuning** of **Llama 3** (`qlora_finetune_llama3.ipynb`) for domain-specific technical question answering.
   * Local/Containerized inference support via **Ollama / Modelfile** (`Modelfile.txt`, `DEPLOYMENT.md`).
4. **Automated Jira Integration (`jira/`)**:
   * Automated background worker (`jira_worker.py`, `jira_client.py`) that monitors Jira tickets, queries the RAG API, posts automated resolutions, and generates evaluation/resolution reports (`jira_report.py`).
5. **Comprehensive Evaluation**:
   * End-to-end evaluation scripts for both the **Retrieval/Reranking** stage (`evaluate_full.py`) and the **LLM Generation** stage (`evaluate_generation.py`), alongside realistic ticket simulation (`psudo_users.py`, `pseudo_users_jira.py`).

---

## Repository Structure

```text
techqa-incident-resolution-diploma-uniwa/
├── app/
│   ├── Dockerfile                             # Dockerfile for the core RAG API service
│   ├── main.py                                # Main application entry point / API endpoints
│   ├── techqa-reranker-v2-finetune.ipynb      # Notebook for fine-tuning the reranker model
│   ├── reranker_model/
│   │   └── reranker_README.md                 # Instructions for downloading/placing reranker weights
│   └── retrieval/
│       ├── build-bm25-techqa.ipynb            # Building the BM25 index on TechQA
│       ├── ingest.py                          # Document ingestion into ChromaDB
│       ├── evaluate_full.py                   # Retrieval & Reranking evaluation script
│       └── psudo_users.py                     # User query simulation for testing
├── chroma_data/
│   └── chroma_README.md                       # Setup notes for ChromaDB persistent storage
├── data/
│   └── data_README.md                         # Instructions for obtaining the TechQA dataset
├── finetuning/
│   ├── build_sft_dataset.py                   # Script to format TechQA for Supervised Fine-Tuning
│   ├── qlora_finetune_llama3.ipynb            # QLoRA fine-tuning notebook for Llama 3
│   ├── evaluate_generation.py                 # Metrics & evaluation for LLM responses
│   ├── Modelfile.txt                          # Ollama Modelfile configuration
│   └── DEPLOYMENT.md                          # Guide for deploying the fine-tuned LLM
├── jira/
│   ├── jira_client.py                         # Jira REST API client wrapper
│   ├── jira_worker.py                         # Polling worker for automated ticket resolution
│   ├── jira_report.py                         # Reporting utility for resolved Jira incidents
│   └── pseudo_users_jira.py                   # Automated ticket generator for Jira testing
├── .env-example                               # Template for environment variables
├── docker-compose.yml                         # Multi-container orchestration
├── requirements.txt                           # Python dependencies
└── LICENSE                                    # License information

### 1. Prerequisites

- Python 3.10+
- Docker & Docker Compose
- Ollama (or compatible LLM inference server, see [`finetuning/DEPLOYMENT.md`](finetuning/DEPLOYMENT.md))
- Jira Account & API Token (for ticketing integration)

### 2. Environment Configuration

Clone the repository and create your `.env` file from the provided template:

```bash
git clone https://github.com/yiannisman/techqa-incident-resolution-diploma-uniwa.git
cd techqa-incident-resolution-diploma-uniwa
cp .env-example .env
```

Update `.env` with your Jira credentials, model endpoints, and database paths.

### 3. Data, Vector DB & Model Weights

Due to size constraints, dataset files, vector indices, and model weights are not tracked directly in Git. Follow the instructions inside each directory's README:

1. **Dataset:** See [`data/data_README.md`](data/data_README.md) to place the TechQA dataset files.
2. **ChromaDB:** See [`chroma_data/chroma_README.md`](chroma_data/chroma_README.md) or run `python app/retrieval/ingest.py` to build the vector store.
3. **Reranker:** See [`app/reranker_model/reranker_README.md`](app/reranker_model/reranker_README.md) to download or place the fine-tuned reranker weights.
4. **LLM Setup:** Follow [`finetuning/DEPLOYMENT.md`](finetuning/DEPLOYMENT.md) to import the fine-tuned Llama 3 model using [`finetuning/Modelfile.txt`](finetuning/Modelfile.txt).

### 4. Running with Docker Compose

Start the application stack using Docker Compose:

```bash
docker-compose up --build -d
```

### 5. Local Development / Running Components Manually

Install the required dependencies:

```bash
pip install -r requirements.txt
```

- **Start the API server:**

  ```bash
  python app/main.py
  ```

- **Run the Jira Worker:**

  ```bash
  python jira/jira_worker.py
  ```

- **Simulate Jira Tickets (Testing):**

  ```bash
  python jira/pseudo_users_jira.py
  ```

## Evaluation

To reproduce the experimental results presented in the thesis:

- **Evaluate Retrieval & Reranking** (Hit Rate, MRR, nDCG):

  ```bash
  python app/retrieval/evaluate_full.py
  ```

- **Evaluate Answer Generation** (ROUGE, BLEU, Semantic Similarity):

  ```bash
  python finetuning/evaluate_generation.py
  ```

## Author & Acknowledgments

- **Author:** IOANNIS MANTZAKOS (`21390126`)
- **Institution:** University of West Attica (UniWA) — Department of Informatics and Computer Engineering

## License

This project is licensed under the terms specified in the [LICENSE](LICENSE) file.

# Αυτοματοποιημένη Επίλυση Περιστατικών IT με Hybrid RAG & Fine-Tuned LLMs (ΕΛΛΗΝΙΚΑ)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED)
![Llama3](https://img.shields.io/badge/LLM-Llama--3%20(QLoRA)-orange)
![ChromaDB](https://img.shields.io/badge/VectorDB-Chroma-green)
![Jira](https://img.shields.io/badge/Integration-Jira%20API-0052CC)

> **Διπλωματική Εργασία** | Τμήμα Μηχανικών Πληροφορικής και Υπολογιστών, Πανεπιστήμιο Δυτικής Αττικής (ΠΑΔΑ)

## Επισκόπηση

Το αποθετήριο περιέχει ένα ολοκληρωμένο (end-to-end) σύστημα βασισμένο σε τεχνητή νοημοσύνη για την **αυτοματοποιημένη επίλυση περιστατικών IT και την τεχνική υποστήριξη**, με χρήση του συνόλου δεδομένων **IBM TechQA**. Το σύστημα συνδυάζει **Hybrid Retrieval-Augmented Generation (RAG)**, έναν προσαρμοσμένο **fine-tuned Cross-Encoder Reranker** και ένα μοντέλο **Llama 3 εκπαιδευμένο με QLoRA**, ενώ ενσωματώνεται πλήρως με το **Jira** ώστε να επεξεργάζεται, να απαντά και να καταγράφει αυτόματα αιτήματα τεχνικής υποστήριξης.

---

## Βασικά Χαρακτηριστικά & Αρχιτεκτονική

1. **Υβριδική Διαδικασία Ανάκτησης (`app/retrieval/`)**:
   * **Sparse Retrieval**: Λεξική αναζήτηση με **BM25** (`build-bm25-techqa.ipynb`).
   * **Dense Retrieval**: Σημασιολογική αναζήτηση διανυσμάτων με **ChromaDB** (`chroma_data/`, `ingest.py`).
2. **Προσαρμοσμένος Fine-Tuned Reranker (`app/reranker_model/`)**:
   * Fine-tuned μοντέλο cross-encoder (`techqa-reranker-v2-finetune.ipynb`) που επαναβαθμολογεί τα ανακτημένα έγγραφα, ώστε να μεγιστοποιείται η συνάφεια του context πριν από την παραγωγή απάντησης.
3. **Fine-Tuning & Ανάπτυξη του LLM (`finetuning/`)**:
   * Κατασκευή συνόλου δεδομένων για Supervised Fine-Tuning (SFT) (`build_sft_dataset.py`).
   * **QLoRA Fine-Tuning** του **Llama 3** (`qlora_finetune_llama3.ipynb`) για απαντήσεις σε τεχνικές ερωτήσεις του συγκεκριμένου πεδίου.
   * Υποστήριξη τοπικού/containerized inference μέσω **Ollama / Modelfile** (`Modelfile.txt`, `DEPLOYMENT.md`).
4. **Αυτοματοποιημένη Ενσωμάτωση με Jira (`jira/`)**:
   * Background worker (`jira_worker.py`, `jira_client.py`) που παρακολουθεί τα tickets του Jira, υποβάλλει ερωτήματα στο RAG API, δημοσιεύει αυτόματες λύσεις και παράγει αναφορές αξιολόγησης/επίλυσης (`jira_report.py`).
5. **Ολοκληρωμένη Αξιολόγηση**:
   * Scripts αξιολόγησης τόσο για το στάδιο **Ανάκτησης/Reranking** (`evaluate_full.py`) όσο και για το στάδιο **Παραγωγής Απαντήσεων από το LLM** (`evaluate_generation.py`), μαζί με ρεαλιστική προσομοίωση tickets (`psudo_users.py`, `pseudo_users_jira.py`).

---

## Δομή Αποθετηρίου

```text
techqa-incident-resolution-diploma-uniwa/
├── app/
│   ├── Dockerfile                             # Dockerfile για την κεντρική υπηρεσία RAG API
│   ├── main.py                                # Κύριο σημείο εκκίνησης εφαρμογής / API endpoints
│   ├── techqa-reranker-v2-finetune.ipynb      # Notebook για το fine-tuning του reranker
│   ├── reranker_model/
│   │   └── reranker_README.md                 # Οδηγίες λήψης/τοποθέτησης των βαρών του reranker
│   └── retrieval/
│       ├── build-bm25-techqa.ipynb            # Κατασκευή του ευρετηρίου BM25 για το TechQA
│       ├── ingest.py                          # Εισαγωγή εγγράφων στη ChromaDB
│       ├── evaluate_full.py                   # Script αξιολόγησης ανάκτησης & reranking
│       └── psudo_users.py                     # Προσομοίωση ερωτημάτων χρηστών για δοκιμές
├── chroma_data/
│   └── chroma_README.md                       # Σημειώσεις ρύθμισης της μόνιμης αποθήκευσης ChromaDB
├── data/
│   └── data_README.md                         # Οδηγίες απόκτησης του συνόλου δεδομένων TechQA
├── finetuning/
│   ├── build_sft_dataset.py                   # Μορφοποίηση του TechQA για Supervised Fine-Tuning
│   ├── qlora_finetune_llama3.ipynb            # Notebook QLoRA fine-tuning για το Llama 3
│   ├── evaluate_generation.py                 # Μετρικές & αξιολόγηση απαντήσεων του LLM
│   ├── Modelfile.txt                          # Ρυθμίσεις Modelfile για το Ollama
│   └── DEPLOYMENT.md                          # Οδηγός ανάπτυξης του fine-tuned LLM
├── jira/
│   ├── jira_client.py                         # Wrapper για το Jira REST API
│   ├── jira_worker.py                         # Polling worker για αυτόματη επίλυση tickets
│   ├── jira_report.py                         # Εργαλείο αναφορών για τα επιλυμένα περιστατικά Jira
│   └── pseudo_users_jira.py                   # Αυτόματη δημιουργία tickets για δοκιμές στο Jira
├── .env-example                               # Πρότυπο μεταβλητών περιβάλλοντος
├── docker-compose.yml                         # Ενορχήστρωση πολλαπλών containers
├── requirements.txt                           # Εξαρτήσεις Python
└── LICENSE                                    # Πληροφορίες άδειας χρήσης
```

---

## Ξεκινώντας

### 1. Προαπαιτούμενα

- Python 3.10+
- Docker & Docker Compose
- Ollama (ή συμβατός διακομιστής LLM inference, βλ. [`finetuning/DEPLOYMENT.md`](finetuning/DEPLOYMENT.md))
- Λογαριασμός Jira & API Token (για την ενσωμάτωση με το σύστημα tickets)

### 2. Ρύθμιση Περιβάλλοντος

Κάντε clone το αποθετήριο και δημιουργήστε το αρχείο `.env` από το παρεχόμενο πρότυπο:

```bash
git clone https://github.com/yiannisman/techqa-incident-resolution-diploma-uniwa.git
cd techqa-incident-resolution-diploma-uniwa
cp .env-example .env
```

Ενημερώστε το `.env` με τα διαπιστευτήρια του Jira, τα endpoints των μοντέλων και τις διαδρομές των βάσεων δεδομένων.

### 3. Δεδομένα, Vector DB & Βάρη Μοντέλων

Λόγω μεγέθους, τα αρχεία δεδομένων, τα ευρετήρια διανυσμάτων και τα βάρη των μοντέλων δεν παρακολουθούνται απευθείας στο Git. Ακολουθήστε τις οδηγίες στο README κάθε φακέλου:

1. **Σύνολο δεδομένων:** Δείτε το [`data/data_README.md`](data/data_README.md) για την τοποθέτηση των αρχείων του TechQA.
2. **ChromaDB:** Δείτε το [`chroma_data/chroma_README.md`](chroma_data/chroma_README.md) ή εκτελέστε `python app/retrieval/ingest.py` για να δημιουργήσετε τη vector βάση.
3. **Reranker:** Δείτε το [`app/reranker_model/reranker_README.md`](app/reranker_model/reranker_README.md) για λήψη ή τοποθέτηση των βαρών του fine-tuned reranker.
4. **Ρύθμιση LLM:** Ακολουθήστε το [`finetuning/DEPLOYMENT.md`](finetuning/DEPLOYMENT.md) για να εισαγάγετε το fine-tuned μοντέλο Llama 3 μέσω του [`finetuning/Modelfile.txt`](finetuning/Modelfile.txt).

### 4. Εκτέλεση με Docker Compose

Εκκινήστε τις υπηρεσίες της εφαρμογής με Docker Compose:

```bash
docker-compose up --build -d
```

### 5. Τοπική Ανάπτυξη / Χειροκίνητη Εκτέλεση Επιμέρους Υπηρεσιών

Εγκαταστήστε τις απαιτούμενες εξαρτήσεις:

```bash
pip install -r requirements.txt
```

- **Εκκίνηση του API server:**

  ```bash
  python app/main.py
  ```

- **Εκτέλεση του Jira Worker:**

  ```bash
  python jira/jira_worker.py
  ```

- **Προσομοίωση Jira Tickets (Δοκιμές):**

  ```bash
  python jira/pseudo_users_jira.py
  ```

## Αξιολόγηση

Για την αναπαραγωγή των πειραματικών αποτελεσμάτων της διπλωματικής:

- **Αξιολόγηση Ανάκτησης & Reranking** (Hit Rate, MRR, nDCG):

  ```bash
  python app/retrieval/evaluate_full.py
  ```

- **Αξιολόγηση Παραγωγής Απαντήσεων** (ROUGE, BLEU, Semantic Similarity):

  ```bash
  python finetuning/evaluate_generation.py
  ```

## Συγγραφέας & Ευχαριστίες

- **Συγγραφέας:** ΙΩΑΝΝΗΣ ΜΑΝΤΖΑΚΟΣ (`21390126`)
- **Ίδρυμα:** Πανεπιστήμιο Δυτικής Αττικής (ΠΑΔΑ) — Τμήμα Μηχανικών Πληροφορικής και Υπολογιστών

## Άδεια Χρήσης

Το έργο διατίθεται υπό τους όρους που ορίζονται στο αρχείο [LICENSE](LICENSE).
