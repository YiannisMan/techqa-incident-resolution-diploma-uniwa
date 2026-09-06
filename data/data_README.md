# Φάκελος `data/`
 
Ο φάκελος αυτός φιλοξενεί τα **μεγάλα δεδομένα, τους δείκτες και τα αρχεία αποτελεσμάτων** του συστήματος. Το περιεχόμενό του **δεν παρακολουθείται από το git** (βλ. `.gitignore`), είτε λόγω μεγέθους είτε λόγω αδειοδότησης. Παρακάτω περιγράφεται τι τοποθετείται εδώ και πώς παράγεται.
 
> Ο φάκελος προσαρτάται στα containers ως `./data:/data` και επιβιώνει των ανακατασκευών (recreate).
>
> **Σημείωση:** η βάση **ChromaDB δεν βρίσκεται εδώ** — αποθηκεύεται σε ξεχωριστό φάκελο `chroma_db/` στη **ρίζα** του repository (στο ίδιο επίπεδο με `app/`, `finetuning/`, `jira/`). Είναι κι αυτός μεγάλος και εξαιρείται από το git.
 
---
 
## 1. Σύνολο δεδομένων TechQA (απαιτείται)
 
Το σύστημα βασίζεται στο σύνολο **IBM TechQA**, το οποίο **δεν περιλαμβάνεται** στο repository καθώς ανήκει στην IBM και διέπεται από δική του άδεια χρήσης.
 
- Επίσημη πηγή / πρόσβαση βλ. αναφορές [7], [12] της διπλωματικής
```
data/
└── TechQA.tar              # το σύνολο TechQA
```
 
---
 
## 2. Δείκτης BM25 (παράγεται σε Kaggle)
 
Λόγω του μεγέθους και των απαιτήσεων μνήμης, ο δείκτης BM25 **δεν χτίζεται τοπικά** αλλά σε περιβάλλον Kaggle. Η ροή έχει δύο βήματα:
 
**α) Tokens (είσοδος):**
```
data/
└── bm25_tokens.jsonl       #  tokenized technotes (id, tokens, doc, meta)
```
Παράγονται από το βήμα προεπεξεργασίας/tokenization των technotes του TechQA.
 
**β) Δείκτης (έξοδος του notebook):** το `build-bm25-techqa.ipynb` (περιλαμβάνεται στο repo) διαβάζει το `bm25_tokens.jsonl` και παράγει:
```
data/
├── bm25s_index/            # ο δείκτης bm25s (vocab.index.json, *.csc.index.npy, params.index.json)
└── bm25_index.pkl          # 2,59 GB - ids, docs, metas (αντιστοίχιση εγγράφων)
```
 
Τα 801.998 έγγραφα ευρετηριάζονται στο Kaggle και μεταφέρονται στο container:
```bash
# από το output του Kaggle notebook, μετά το κατέβασμα:
docker cp bm25s_index knowledge_api:/data/bm25s_index
docker cp bm25_index.pkl knowledge_api:/data/bm25_index.pkl
docker restart knowledge_api
```
 
---
 
## 3. Σύνολο εκπαίδευσης SFT (Στάδιο Β)
 
```
data/
└── sft_dataset.jsonl       # τα 450 παραδείγματα εκπαίδευσης (MODE-A / MODE-B)
```
Παράγεται από το `finetuning/build_sft_dataset.py` και τροφοδοτεί την εξειδίκευση QLoRA.
 
---
 
## 4. Αρχεία αποτελεσμάτων & αξιολόγησης (όλες οι φάσεις)
 
Στο `data/` αποθηκεύονται και τα CSV αξιολόγησης/αποτελεσμάτων από όλα τα στάδια. Παράγονται από τον κώδικα και δεν χρειάζεται να ανέβουν (προαιρετικά ως τεκμήριο):
 
```
data/
├── eval_results.csv    # Στάδιο Α - μετρικές ανάκτησης (recall@K, MRR)
├── generation_eval_results.csv   # Στάδιο Β - COMPARISON ACROSS K / GATE ANALYSIS
├── jira_ticket_map.csv     # Στάδιο Γ - qid, issue_key, answerable, gt_document
└── jira_worker_log.csv     # Στάδιο Γ - issue_key, escalated, reason, gated, cited_ids, ...
```
 
> **Προσοχή:** το `jira_ticket_map.csv` περιέχει το `gt_document` του TechQA. Αν δημοσιεύσετε αποτελέσματα, προτιμήστε μόνο το **συγκεντρωτικό output** του `jira_report.py`, όχι το πλήρες map.
>
> **Μην ανοίγετε τα CSV με Excel** - προσθέτει επιπλέον γραμμές header/BOM/semicolons που χαλούν την ανάγνωση. Χρησιμοποιήστε `head`/`cat` ή text editor.
 
---
