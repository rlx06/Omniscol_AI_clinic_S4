# Omniscol Translation Pipeline: AI Clinic (Semester 4)

An LLM-powered pipeline that generates, validates and maintains the multilingual UI translations (~1,200+ keys, 14 languages) for **Omniscol**, a school-scheduling platform. It was built during the AI Clinic at aivancity (Semester 4, 2026).

> This is **Version 1** (chunk-based translation). The improved, concept-aware **Version 2**, built during the follow-up internship, has semantic batching, translation memory, 35 unit tests and CI. It lives in **[omniscol-translation-pipeline-v2-INTERNSHIP](https://github.com/rlx06/omniscol-translation-pipeline-v2-INTERNSHIP)**.

## Quick start

```bash
pip install -r requirements.txt
export OMNISCOL_TOKEN=...        # API token (never commit it)
gcloud auth application-default login   # Vertex AI / Gemini access

python scripts/01_fetch_from_api.py --lang fr                         # pull the French reference files
python scripts/03_translation_pipeline.py generate-language pl Polish  # translate a whole language
python scripts/03_translation_pipeline.py sync-missing-keys --map '{"pl": "Polish"}'
python scripts/03_translation_pipeline.py audit                        # audit report
```

| Folder | Contents |
|---|---|
| `data/source/` | French reference JSON plus the glossary (source of truth) |
| `languages/` | Generated translation files, one per language and module |
| `reports/` | Validation and audit reports |
| `scripts/` | Fetch and translation pipeline scripts |

---

PIPELINE EXPLANATION


## Translation Pipeline Design

The translation pipeline is designed as a modular command-line system that automates the generation, validation, and maintenance of multilingual JSON translation files.

### Step 1: Data Source

The French translation files (`fr_webapp.json` and `fr_login.json`) act as the reference source of truth. All other languages are derived from these files.

---

### Step 2: Chunk-Based Processing

Since translation files contain more than 1200 keys, the pipeline splits them into smaller chunks.

This ensures:
- reliable AI processing
- reduced API failure risk
- easier validation

---

### Step 3: AI-Based Translation

Each chunk is sent to Gemini (via Vertex AI) with:
- strict translation instructions
- glossary constraints
- formatting preservation rules

The model returns translated JSON values while keeping keys unchanged.

---

### Step 4: Validation Layer

After each translation, the pipeline performs validation checks:

- Key consistency (no missing or extra keys)
- Placeholder preservation (`{name}`, `%s`, etc.)
- HTML tag consistency (`<br>`, `<b>`, etc.)
- Detection of untranslated values

If validation fails, the chunk is retried automatically.

---

### Step 5: Retry and Error Handling

The system includes:
- retry mechanism (up to 5 attempts)
- exponential backoff
- detection of retryable errors (e.g., 429, timeouts)

This improves robustness when interacting with external APIs.

---

### Step 6: Language File Generation

Validated chunks are merged into final translation files:
- `<lang>_webapp.json`
- `<lang>_login.json`

---

### Step 7: Sync Missing Keys

To maintain consistency over time:
- missing keys are detected by comparing with the French reference
- only missing entries are translated and inserted

This avoids full regeneration and reduces API usage.

---

### Step 8: Audit System

An audit module analyzes existing translations and generates reports highlighting:
- inconsistent terms
- glossary violations
- potential translation issues

Reports are generated in Markdown format for easy review.

---

### Overall Pipeline Flow

French Source → Chunking → AI Translation → Validation → Merge → Output JSON  
                                                                                 ↓  
                                                                        Audit / Sync

---

This architecture ensures scalability, consistency, and partial automation while maintaining cont
