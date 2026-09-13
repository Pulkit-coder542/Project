# VERITAS

### Agentic AI Multi-Document Investigation Platform

VERITAS is a multi-document investigation platform designed to help organizations identify contradictions, trace evidence, detect missing supporting documentation, and produce explainable investigation findings.

Unlike traditional document question-answering systems, VERITAS is designed around **investigation rather than retrieval**. It combines information from multiple documents, builds relationships between extracted claims, identifies inconsistencies, and highlights evidence gaps that require human verification.

> **Current Status:** Prototype / Hackathon Development
>
> **Current Phase:** Phase 5.1 — Deterministic Investigation Engine

---

# 1. Problem

Organizations and individuals make important decisions using information distributed across:

- Contracts
- Purchase Orders
- Invoices
- Emails
- Reports
- Policies
- Amendments
- Other business documents

Manually reviewing these documents makes it difficult to:

- Identify contradictions across documents
- Determine which records support each other
- Detect missing authorization or supporting evidence
- Trace findings back to their original evidence
- Understand how multiple documents collectively support a conclusion

Traditional document Q&A systems generally answer questions from retrieved text.

VERITAS instead focuses on:

> **What can be proven from the available documents, what conflicts, and what evidence is missing?**

---

# 2. Core Concept

Traditional RAG:

```text
User Query
    ↓
Document Retrieval
    ↓
Answer
```

VERITAS:

```text
Documents
    ↓
Text Extraction
    ↓
Claims / Facts
    ↓
Evidence
    ↓
Relationships
    ↓
Investigation
    ↓
Contradictions + Evidence Gaps
    ↓
Findings
    ↓
Human Verification
    ↓
Investigation Report
```

The current implementation contains the deterministic portions of this workflow. The AI-assisted reasoning layer is planned for a later phase.

---

# 3. Current Features

## Document Management

The current application supports:

- Multiple document selection
- Demo documents
- Local file uploads
- PDF
- DOCX
- TXT
- CSV
- XLSX
- PNG
- JPG / JPEG
- File metadata
- Document selection and removal
- IndexedDB persistence for uploaded document blobs
- Backend document storage and metadata

## Document Processing

| File Type | Current Processing |
| --- | --- |
| PDF | Text extraction using PyMuPDF |
| DOCX | Text extraction using python-docx |
| TXT | Standard text reading |
| CSV | Standard CSV parsing |
| XLSX | openpyxl extraction |
| PNG | Stored; OCR not currently implemented |
| JPG / JPEG | Stored; OCR not currently implemented |

Image documents are marked as requiring OCR rather than pretending OCR has been performed.

---

# 4. Structured Intelligence Layer

The current pipeline is:

```text
Document
    ↓
Extracted Text
    ↓
Content-Based Claim Extraction
    ↓
Claim
    ↓
Evidence
    ↓
Investigation Context
```

Claims currently contain:

- Claim ID
- Document ID
- Claim text
- Normalized value
- Claim type
- Confidence
- Evidence text
- Source reference

## Current Claim Types

- `CONTRACT_VALUE`
- `APPROVED_AMOUNT`
- `INVOICE_TOTAL`
- `REVISED_PRICE`

Extraction is currently deterministic, rule-based, and content-driven rather than LLM-based.

---

# 5. Investigation Engine

The current investigation engine is deterministic.

It receives an `InvestigationContext` containing claims and expected evidence and produces an `InvestigationResult`.

```text
InvestigationContext
        ↓
Deterministic Investigation Engine
        ↓
InvestigationResult
   ┌────┴───────────────┐
   ↓                    ↓
Findings          Relationships
   ↓                    ↓
Evidence Gaps       Summary
```

## Relationship Types

The investigation model supports:

- `SUPPORTS`
- `CONTRADICTS`
- `CORROBORATES`
- `DEPENDS_ON`
- `RELATED_TO`

The current engine generates `CORROBORATES` relationships when comparable claims have matching normalized values.

---

# 6. Evidence Gaps

VERITAS distinguishes an actual document from evidence that is expected but missing.

Example:

```text
Expected Evidence
└── Signed Contract Amendment #1
    └── Status: MISSING
```

The missing amendment is not represented as an actual document.

This prevents the system from falsely implying that a missing document was found or processed.

---

# 7. Golden Investigation Case

The prototype uses a procurement/finance scenario.

| Document | Claim |
| --- | --- |
| Contract_v1.pdf | Contract value: ₹10,00,000 |
| PO_4821.pdf | Approved amount: ₹12,00,000 |
| Invoice_7732.pdf | Invoice total: ₹12,00,000 |
| Email_March12.pdf | Revised price: ₹12,00,000 |
| Signed Amendment #1 | Missing |

The engine identifies:

```text
Contract baseline
₹10,00,000
       ↓
       └── differs from ──→ ₹12,00,000

PO ───────────────┐
Invoice ──────────┼── CORROBORATE → ₹12,00,000
Email ────────────┘

Missing:
Signed Contract Amendment #1
```

## Current Finding

Revised contract value lacks formal authorization evidence

| Property | Value |
| --- | --- |
| Severity | HIGH |
| Status | REQUIRES_HUMAN_VERIFICATION |
| Confidence | 0.90 |

The finding does not claim fraud, misconduct, or intent.

Instead, it identifies an evidence/control gap:

The original contract records ₹10,00,000 while downstream records consistently reference ₹12,00,000, but formal amendment evidence is missing.

Recommended action:

Request and verify the executed amendment supporting the revised value.

---

# 8. Investigation Logic

The engine checks whether downstream records agree before describing them as corroborating evidence.

## Consistent Downstream Evidence

```text
Contract:  ₹10L
PO:        ₹12L
Invoice:   ₹12L
Email:     ₹12L

Result:
✓ PO ↔ Invoice corroborate
✓ PO ↔ Email corroborate
✓ Invoice ↔ Email corroborate
✓ Contract differs from downstream value
✓ Authorization-gap finding generated
```

## Inconsistent Downstream Evidence

```text
Contract:  ₹10L
PO:        ₹12L
Invoice:   ₹15L

Result:
✓ Difference detected
✗ No false corroboration
✗ No golden authorization-gap finding
✓ Investigation reports inconsistent downstream values
```

This prevents the engine from incorrectly describing conflicting evidence as mutually corroborating.

---

# 9. Human Verification

VERITAS uses a human-in-the-loop model.

```text
Evidence
    ↓
System Finding
    ↓
Human Verification
    ↓
Final Resolution
```

High-severity findings are not automatically treated as resolved.

---

# 10. Application Structure

```text
VERITAS/
│
├── app/
│   ├── documents/
│   ├── findings/
│   ├── investigations/
│   │   ├── [id]/
│   │   └── new/
│   ├── reports/
│   ├── settings/
│   └── page.tsx
│
├── backend/
│   └── app/
│       ├── config.py
│       ├── main.py
│       └── schemas.py
│
├── components/
│   ├── ui/
│   ├── app-shell.tsx
│   ├── confidence-indicator.tsx
│   ├── empty-state.tsx
│   ├── evidence-reference.tsx
│   ├── page-header.tsx
│   ├── severity-badge.tsx
│   └── status-badge.tsx
│
├── features/
│   └── investigations/
│       ├── components/
│       ├── demo-data.ts
│       ├── demo-session.ts
│       └── engine.ts
│
├── lib/
│   ├── api.ts
│   ├── db.ts
│   └── utils.ts
│
├── types/
│   ├── engine.ts
│   ├── intelligence.ts
│   └── investigation.ts
│
├── .gitignore
├── package.json
├── pnpm-lock.yaml
├── pnpm-workspace.yaml
├── next.config.ts
└── tsconfig.json
```

---

# 11. Technology Stack

## Frontend

- Next.js
- React
- TypeScript
- Tailwind CSS
- shadcn/ui-compatible components

## Backend

- FastAPI
- Python
- Pydantic
- Uvicorn

## Document Processing

- PyMuPDF
- python-docx
- openpyxl
- Python standard library parsers

## Local Storage

- Browser IndexedDB for uploaded document blobs
- Local backend storage for processed documents
- In-memory backend metadata in the current prototype

---

# 12. Backend API

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/api/health` | Backend health check |
| POST | `/api/documents/upload` | Upload documents |
| GET | `/api/documents` | List documents |
| GET | `/api/documents/{id}` | Get document metadata |
| GET | `/api/documents/{id}/content` | Get extracted content |
| POST | `/api/documents/{id}/extract` | Extract claims |

---

# 13. Current Architecture

```text
┌──────────────────────────────┐
│          Next.js UI          │
│                              │
│ Investigation Setup          │
│ Document Management          │
│ Investigation Workspace      │
│ Findings                     │
│ Reports                      │
└──────────────┬───────────────┘
               │
               │ API
               ▼
┌──────────────────────────────┐
│         FastAPI Backend      │
│                              │
│ Document Upload              │
│ File Storage                 │
│ Text Extraction              │
│ Claim Extraction             │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│    Structured Intelligence   │
│                              │
│ Claims                       │
│ Evidence                     │
│ Expected Evidence            │
│ Investigation Context        │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ Deterministic Engine         │
│                              │
│ Relationships                │
│ Contradictions               │
│ Findings                     │
│ Evidence Gaps                │
│ Investigation Summary        │
└──────────────────────────────┘
```

---

# 14. Development Phases

| Phase | Status |
| --- | --- |
| Phase 1 — Application Foundation | Complete |
| Phase 2 — Investigation Workflow | Complete |
| Phase 2.5 — Local File Upload | Complete |
| Phase 3 — Backend Ingestion | Complete |
| Phase 4 — Structured Intelligence | Complete |
| Phase 5 — Investigation Engine | Complete |
| Phase 5.1 — Investigation Hardening | Complete |
| Phase 6 — AI-Assisted Investigation | Planned |

## Phase 1 — Application Foundation

- Next.js/TypeScript foundation
- Application shell
- Dashboard
- Core routes
- Shared UI components
- Typed domain models
- Investigation engine abstraction
- Golden-path demo data

## Phase 2 — Investigation Workflow

- Investigation creation
- Investigation name and goal
- Document selection
- Review step
- Deterministic preparation sequence
- Local persistence
- Investigation register
- Workspace navigation

## Phase 2.5 — Local File Upload

- Multi-file upload
- Supported file types
- File metadata
- Document selection/removal
- IndexedDB storage
- Demo and uploaded documents coexist

## Phase 3 — Backend Ingestion

- FastAPI backend
- Multipart file upload
- Local document storage
- File metadata
- Text extraction
- Frontend/backend integration
- Content retrieval endpoint

## Phase 4 — Structured Intelligence

- Claims
- Evidence
- Expected evidence
- Investigation context
- Deterministic claim extraction
- Content-based extraction

## Phase 5 — Investigation Engine

- Deterministic investigation engine
- Findings
- Evidence gaps
- Investigation summaries
- Relationship model
- Human verification status
- Golden investigation logic

## Phase 5.1 — Investigation Hardening

- Corroboration relationships
- Downstream consistency validation
- Improved inconsistent-evidence handling
- Empty investigation status handling
- Golden-case validation scenarios

---

# 15. Installation Requirements

Before running VERITAS, install the following:

| Requirement | Purpose |
| --- | --- |
| Git | Version control |
| Node.js | Runs the Next.js frontend |
| pnpm | Frontend package management |
| Python 3.x | Runs the FastAPI backend |
| pip | Installs Python dependencies |

## Recommended Versions

The project was developed using modern Node.js and pnpm.

Check your installed versions:

```bash
node --version
npm --version
pnpm --version
python --version
pip --version
git --version
```

---

# 16. Getting Started

## Step 1 — Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/veritas.git
```

Enter the project directory:

```bash
cd veritas
```

## Step 2 — Install Frontend Dependencies

```bash
pnpm install
```

This installs the packages listed in `package.json`.

## Step 3 — Configure Frontend Environment

Create a file named:

`.env.local`

Add:

```env
NEXT_PUBLIC_API_URL=http://localhost:8000
```

`.env.local` should not be committed to GitHub.

## Step 4 — Set Up the Backend

Create a Python virtual environment:

```bash
python -m venv .venv
```

Windows:

```powershell
.venv\Scripts\activate
```

macOS / Linux:

```bash
source .venv/bin/activate
```

## Step 5 — Install Backend Dependencies

From the project root:

```bash
pip install -r backend/requirements.txt
```

---

# 17. Running VERITAS

VERITAS currently requires two processes:

- Next.js frontend
- FastAPI backend

## Terminal 1 — Start Backend

From the project root:

```bash
uvicorn backend.app.main:app --reload --port 8000
```

The backend should be available at:

http://localhost:8000

Health check:

http://localhost:8000/api/health

## Terminal 2 — Start Frontend

Open another terminal.

From the project root:

```bash
pnpm dev
```

The frontend should be available at:

http://localhost:3000

Open:

http://localhost:3000

---

# 18. Running the Demo

Once both servers are running:

1. Open the frontend at http://localhost:3000
2. Navigate to New Investigation
3. Enter an investigation name and goal
4. Select the demo documents
5. Review the selected documents
6. Start the investigation
7. Open the investigation workspace
8. Review extracted claims
9. Review evidence relationships
10. Review detected findings
11. Review missing evidence
12. Review the recommended human-verification action

The golden demonstration focuses on the difference between:

Contract Value: ₹10,00,000

and:

PO / Invoice / Email: ₹12,00,000

with the expected:

Signed Contract Amendment #1

missing.

---

# 19. Development Commands

## Frontend

Install dependencies:

```bash
pnpm install
```

Start development server:

```bash
pnpm dev
```

Run lint:

```bash
pnpm lint
```

Run TypeScript checking:

```bash
pnpm exec tsc --noEmit
```

Create production build:

```bash
pnpm build
```

## Backend

Start FastAPI:

```bash
uvicorn backend.app.main:app --reload --port 8000
```

---

# 20. Current Limitations

VERITAS is currently a prototype.

Not currently implemented:

- LLM-based investigation reasoning
- LangGraph agent orchestration
- Vector database
- PostgreSQL
- Redis
- Celery
- OCR
- Web search / external evidence retrieval
- Production authentication
- Production authorization
- Persistent backend database
- Advanced natural-language inference
- Production-scale document processing

The current investigation engine is deterministic and rule-based.

---

# 21. Design Principles

## Evidence First

Every finding should be grounded in available evidence.

## No Fabricated Evidence

The system should never imply that an unavailable document or source was processed.

## Human Verification

The system identifies potential issues; humans make final decisions.

## Explainability

Findings should be traceable to claims and their supporting evidence.

## Conservative Reasoning

Document inconsistencies should not automatically be interpreted as fraud or intentional wrongdoing.

## Deterministic Grounding

The deterministic investigation layer provides a predictable foundation that future AI reasoning can build upon.

---

# 22. Future Direction

The planned architecture extends the deterministic foundation with AI-assisted reasoning:

```text
Documents
    ↓
Extraction
    ↓
Claims + Evidence
    ↓
Deterministic Relationships
    ↓
AI Investigation
    ↓
Hypothesis Testing
    ↕
Counter-Evidence Search
    ↓
Evidence-Grounded Findings
    ↓
Human Verification
    ↓
Final Report
```

The goal is for the AI layer to reason over structured, traceable evidence rather than independently generating unsupported conclusions.

---

# 23. Project Status

```text
Phase 1       ████████████████████  Complete
Phase 2       ████████████████████  Complete
Phase 2.5     ████████████████████  Complete
Phase 3       ████████████████████  Complete
Phase 4       ████████████████████  Complete
Phase 5       ████████████████████  Complete
Phase 5.1     ████████████████████  Complete
Phase 6       ░░░░░░░░░░░░░░░░░░░░  Planned
```

---

# 24. License

This project is currently being developed as a hackathon project.

License information will be added when the project is finalized.

---

# Quick setup summary

For a teammate who just cloned the repo:

```bash
git clone https://github.com/YOUR_USERNAME/veritas.git
cd veritas
pnpm install
python -m venv .venv
```

Terminal 1:

```powershell
.venv\Scripts\activate
pip install -r backend/requirements.txt
uvicorn backend.app.main:app --reload --port 8000
```

Terminal 2:

```bash
pnpm dev
```

Then open:

http://localhost:3000

One thing I'd do before pushing this README: replace `YOUR_USERNAME/veritas.git` with your actual team's GitHub repository URL once you've created the shared repository.
