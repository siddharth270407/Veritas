# Veritas

> Ask questions about your PDFs. Get answers **with proof**. If the proof is missing, Veritas says **"I don't have proof"** instead of making something up.

Built for **Hackasol 5.0 · Track 2 (AIML) · AIML-02: Zero-Guessing & Contradiction-Checking Document AI**.
Everything runs **on your own computer** with open-source models. No paid AI service is used.

---

## 1. The problem

Banks, insurers and government offices work with long PDFs such as loan agreements, policy terms and circulars.
Normal AI chatbots have three problems with them:

1. They **invent answers** when the document does not say.
2. They **miss conflicts**, for example one page says "2%" and another says "3%".
3. They **don't know which version is current** when a document has been amended several times.

## 2. What Veritas does

| Feature | What it means in simple words |
|---|---|
| **Answers with proof** | Every sentence comes with the document name, page number and the exact quote. |
| **Says "I don't have proof"** | If the PDFs don't contain the answer, it refuses and tells you what is missing. |
| **Finds conflicts** | If two places disagree, it shows both quotes side by side. |
| **Tracks amendments** | If document C changes B, and B changed A, it tells you which rule applies today. |
| **Faithfulness score** | A percentage showing how much of the draft answer was backed by real quotes. |
| **Highlights the source** | Click a citation to see the PDF page with the passage highlighted. |
| **Hindi support** | Ask in Hindi or English. |
| **Calculations** | For example, "late fee on Rs. 20,000". The numbers come from the documents and Python does the maths. |

## 3. How it works (the simple version)

```
  Your PDFs
     |
     v
 1. READ        Split each PDF into small pieces and remember the page number of each piece.
     |
     v
 2. FIND        When you ask a question, search all PDFs for the most relevant pieces.
     |
     v
 3. DRAFT       The AI writes an answer, and must attach an exact quote to every sentence.
     |
     v
 4. CHECK       Our code checks every sentence:
     |            - Is the quote really in the PDF?
     |            - Are all the numbers in the quote?
     |            - Does the quote really prove the sentence?
     |          Sentences that fail are BLOCKED and never shown as an answer.
     v
 5. COMPARE     Look for conflicts, and work out which amendment is newest.
     |
     v
 6. SHOW        Verified answer + quotes + conflicts + amendment timeline
                (or "I don't have proof" if nothing could be verified).
```

**The key idea:** the AI is only allowed to *propose*. Our own code *checks* every claim against the real PDF text.

## 4. Models used (all open source, all local)

| Job | Model | Where it runs |
|---|---|---|
| Writing and checking answers | **Qwen3.5** (`qwen3.5:4b` or `qwen3.5:9b`) | Ollama |
| Understanding meaning (search, Hindi + English) | **bge-m3** | Ollama |
| Keyword search | BM25 (written by us) | Python |
| Reading scanned pages (optional) | Tesseract OCR | Your computer |

## 5. Project structure

```
Veritas/
│
├── README.md              <- you are here
├── requirements.txt       <- Python packages to install
├── run.sh                 <- one command to start the app
│
├── backend/               <- the "brain" of the app (Python)
│   ├── config.py          Settings: model names, thresholds
│   ├── llm.py             Talks to Ollama (the AI models)
│   ├── ingest.py          Reads PDFs: text, tables, page numbers, dates, scanned pages
│   ├── store.py           Saves everything and searches it
│   ├── textutil.py        Helper tools: keyword search, quote checking, number checking
│   ├── qa.py              Question answering + the "zero-guessing" checks + faithfulness score
│   ├── reasoning.py       Finds conflicts and works out which amendment applies
│   ├── highlight.py       Draws the yellow highlight on a PDF page
│   └── main.py            The web server (connects the brain to the screen)
│
├── frontend/
│   └── index.html         <- the web page you see in the browser
│
├── demo_pdfs/             <- sample documents to try the app
│   ├── Loan_Agreement_2023.pdf
│   ├── Amendment_1_2024.pdf
│   ├── Amendment_2_2025.pdf
│   ├── Amendment_3_2027.pdf       (not yet in force: good demo!)
│   └── Product_Brochure_2025.pdf  (contradicts the agreement: good demo!)
│
├── scripts/
│   └── make_demo_pdfs.py  <- recreates the demo PDFs
│
├── tests/
│   └── test_logic.py      <- checks the verification logic works
│
└── data/                  <- created automatically (your uploaded PDFs and the search index)
```

### Which file does what?

| If you want to change... | Open this file |
|---|---|
| Which AI model is used | `backend/config.py` |
| How PDFs are read or split | `backend/ingest.py` |
| How answers are checked | `backend/qa.py` |
| How conflicts are found | `backend/reasoning.py` |
| What the website looks like | `frontend/index.html` |
| The web API | `backend/main.py` |

## 6. How to run it

**Step 1. Install Ollama** from https://ollama.com/download, then download the two models:

```bash
ollama pull qwen3.5:4b      # use qwen3.5:9b if your computer has 16 GB RAM or more
ollama pull bge-m3
```

**Step 2. Install the project:**

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Step 3. Start the app:**

```bash
LLM_MODEL=qwen3.5:4b ./run.sh      # Windows: set LLM_MODEL=qwen3.5:4b  then  uvicorn backend.main:app --port 8000
```

**Step 4. Open** http://localhost:8000

If the top-right badge shows your model names, everything is connected.

## 7. Try it in 2 minutes

1. Upload all five PDFs from `demo_pdfs/`.
2. Ask these questions:

| Question | What you should see |
|---|---|
| What is the late payment charge that applies today? | **2.5%**, with the chain 2% → 3% → 2.5% |
| What is the prepayment charge? | **2%** today. The 0% change is shown as *not yet in force* |
| What is the processing fee for a loan of Rs. 4,00,000? | **1.5%** from the fee table, plus a **conflict** with the brochure (0.5%) |
| What is the foreclosure penalty for a gold loan? | **"I don't have proof"**: it refuses and lists what is missing |
| If an instalment of Rs. 20,000 is overdue for one month, what is the late charge? | A calculation with the quoted inputs shown |

3. Click any citation to see the highlighted PDF page.
4. Open **Conflicts & amendments** and press **Scan all documents for conflicts**.

## 8. What the answer screen means

| Label | Meaning |
|---|---|
| 🟢 **Verified answer** | Every sentence has a real quote that proves it. |
| 🟡 **Partially verified** | Some sentences were blocked, so only the proven ones are shown. |
| 🟠 **Documents conflict** | The PDFs disagree and no amendment settles it, so we show both sides. |
| 🔴 **I don't have proof** | Nothing could be proven, so we refuse to guess. |

## 9. Known limits

- A small laptop model is slow. Expect 1 to 3 minutes per question.
- If the search misses the right page, the app will refuse or answer only partly. It never fills the gap with a guess.
- Amendment links and dates are detected automatically. If one is wrong, use the **Edit** button on the document.
- Handwritten pages are not supported.

## 10. Team

| Name | GitHub | Worked on |
|---|---|---|
| Sarthak Deawangan| @sarthak05012006 | PDF reading and search (`ingest.py`, `store.py`, `textutil.py`) |
| Aditya Singh | @adityasingh13177-lgtm | Answer checking (`qa.py`, `llm.py`) |
| Aryan Chande | @aryanchande | Conflicts, amendments and tests (`reasoning.py`, `tests/`) |
| Siddharth Arya | @siddharth270407 | Web app and documentation (`main.py`, `frontend/`, `README.md`) |

## 11. License

MIT. See `LICENSE`.
