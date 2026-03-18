# 🍹 CocktailMachines — Invoice Automation System

> **AI-powered, zero-touch invoice processing pipeline built on n8n**  
> Automatically ingests, OCRs, classifies, matches, and books invoices — from Gmail to ERP — without human intervention.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Architecture](#-architecture)
- [Workflows](#-workflows)
- [Tech Stack](#-tech-stack)
- [Database Schema](#-database-schema)
- [Folder Structure](#-folder-structure)
- [Setup & Installation](#-setup--installation)
- [Environment & Credentials](#-environment--credentials)
- [Invoice Processing Flow](#-invoice-processing-flow)
- [AI Assistants](#-ai-assistants)
- [Error Handling](#-error-handling)
- [Invoice Status States](#-invoice-status-states)

---

## 🔍 Overview

CocktailMachines B.V. processes dozens of supplier invoices daily. This n8n automation suite eliminates all manual handling by:

1. **Polling Gmail** every minute for new invoice emails
2. **Extracting text** from PDF, JPEG, and UBL XML invoices via OCR (PDF.co) or XML parsing
3. **Resolving the supplier** using an OpenAI Assistant backed by a Vector Store knowledge base
4. **Classifying the invoice** as purchase (`inkoopfactuur`), cost (`kostenfactuur`), or unknown (`onbekend`)
5. **Matching to open purchase orders** from the ERP database (for purchase invoices)
6. **Mapping to general ledger codes** (for cost invoices)
7. **Storing results** in MySQL and archiving files on FTP

Every step is logged — full audit trail, zero data loss.

---

## 🏗 Architecture

```
Gmail Inbox
     │
     ▼
┌─────────────────────────────────────────────┐
│              MAIN FLOW (Orchestrator)        │
│  Email → Parse → OCR/XML → DB Log → Route   │
└────────────────────┬────────────────────────┘
                     │
          ┌──────────▼──────────┐
          │   A1: Supplier      │  ← OpenAI Assistant
          │   Resolver          │     + Vector Store
          └──────────┬──────────┘
                     │
          ┌──────────▼──────────┐
          │   A2: Classifier    │  ← OpenAI Assistant
          │   & Transformer     │     (UBL JSON output)
          └──────────┬──────────┘
                     │
          ┌──────────┴──────────┐
          │                     │
  ┌───────▼───────┐     ┌───────▼───────┐
  │ A3: Purchase  │     │ A4: Ledger    │
  │ Order Matcher │     │ Mapper        │
  │ (inkoopfact.) │     │ (kostenfact.) │
  └───────┬───────┘     └───────┬───────┘
          │                     │
          └──────────┬──────────┘
                     │
          ┌──────────▼──────────┐
          │  MySQL + FTP        │
          │  Final Storage      │
          └─────────────────────┘
```

---

## 🔄 Workflows

| # | Workflow | Trigger | Purpose | AI? |
|---|----------|---------|---------|-----|
| 1 | **Main Flow** | Gmail (every minute) | Orchestrator: email → OCR → routing | GPT-4.1 |
| 2 | **A1 — Supplier Resolver** | Sub-workflow call | Identify supplier from invoice text | ✅ OpenAI Assistant |
| 3 | **A2 — Invoice Classifier** | Sub-workflow call | Classify type + transform to UBL JSON | ✅ OpenAI Assistant |
| 4 | **A3 — PO Matcher** | Sub-workflow call | Match invoice to open purchase orders | ✅ OpenAI Assistant |
| 5 | **A4 — Ledger Mapper** | Sub-workflow call | Assign GL codes to cost invoices | ✅ OpenAI Assistant |
| 6 | **Vector Store Updater** | Manual | Maintain supplier knowledge base in OpenAI | GPT-4.1 |

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| Automation Platform | [n8n](https://n8n.io) (self-hosted) |
| AI / LLM | OpenAI GPT-4.1 + OpenAI Assistants API |
| AI Knowledge Base | OpenAI Vector Store |
| OCR | [PDF.co API](https://pdf.co) |
| Image OCR (fallback) | Mistral Cloud API |
| Primary Database | MySQL (Test environment) |
| ERP Database | MySQL (Production — read-only for PO data) |
| File Storage | FTP Server (`facturen/new/`, `facturen/errors/`) |
| Email | Gmail OAuth2 |
| Invoice Format | UBL 2.1 XML + PDF + JPEG |

---

## 🗄 Database Schema

### `email_logs`
Stores every incoming email received by the Gmail trigger.

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| sender | VARCHAR(255) | Sender email address |
| subject | VARCHAR(255) | Email subject |
| date | DATETIME | Receipt timestamp |
| body | TEXT | Email plain text |
| status | VARCHAR(50) | `RECEIVED` |

### `processed_files`
One record per invoice file — tracks processing stage.

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| filename | VARCHAR(255) | Invoice filename (no extension) |
| status | VARCHAR(50) | See [status states](#-invoice-status-states) |
| email_id | INT FK | → `email_logs.id` |
| ocr_data | LONGTEXT | Extracted OCR text body |
| supplier_id | INT | Resolved supplier (from A1) |
| confidence_score | FLOAT | AI confidence 0.0–1.0 |

### `aws_invoice_json`
Final structured invoice data after full AI processing.

| Column | Type | Description |
|--------|------|-------------|
| id | INT PK | Auto-increment |
| document_id | INT FK | → `processed_files.id` |
| ubl_json | LONGTEXT | UBL-format Invoice JSON |
| validation_json | LONGTEXT | Validation + match results |
| supplier_id | INT | Supplier ID |
| invoice_type | VARCHAR(20) | `cost` / `purchase` / `none` |
| issue_date | DATE | Invoice date |
| total_payable | DECIMAL(10,2) | Invoice total amount |
| currency | VARCHAR(10) | Currency code (EUR) |
| status | VARCHAR(20) | `MATCHED` / `NEW` / `REVIEW` / `REJECTED` |
| updated_at | DATETIME | Last update |

### `settings`
Single-row global settings table.

| Column | Description |
|--------|-------------|
| invoice_counter | Auto-incrementing invoice reference number |
| pdf_credits | Remaining PDF.co API credits |

### `awf_supplier_chunk` *(Production DB)*
Maps each supplier to their OpenAI Vector Store file.

| Column | Description |
|--------|-------------|
| rc_ID | Chunk record ID |
| supplierID | ERP supplier ID |
| vs_file_id | OpenAI file ID |
| vs_filename | Markdown filename (e.g. `leverancier-42.md`) |
| vs_bUpdate | `1` = needs rebuild |
| vs_bDelete | `1` = marked for deletion |

---

## 📁 Folder Structure

```
cocktailmachines-invoice-automation-n8n/
│
├── 📁 cocktailmachines_package/
│   │
│   ├── 📁 flows_json/               ← n8n importable JSON files
│   │   ├── Flow_1_Main_Flow.json
│   │   ├── Flow_A1_Resolve_Invoice_Supplier.json
│   │   ├── Flow_A2_Invoice_Classifier.json
│   │   ├── Flow_A3_Purchase_Order_Matcher.json
│   │   ├── Flow_A4_Ledger_Mapper.json
│   │   ├── Flow_Vector_Store_Updater.json
│   │   └── README.txt
│   │
│   ├── 📁 flow_screenshots/         ← n8n canvas screenshots
│   │   ├── Flow_1_Main_Flow_Overview.png
│   │   ├── Flow_A1_Supplier_Resolver.png
│   │   ├── Flow_A2_Invoice_Classifier.png
│   │   ├── Flow_A3_PO_Matcher.png
│   │   ├── Flow_A4_Ledger_Mapper.png
│   │   └── Flow_5_Vector_Store_Updater.png
│   │
│   └── 📁 documentation/
│       └── CocktailMachines_n8n_Documentation.pdf   ← Full 28-page technical doc
│
└── README.md
```

---

## ⚙ Setup & Installation

### Prerequisites
- n8n instance (v1.0+) — self-hosted or cloud
- MySQL database (test + production)
- OpenAI account with Assistants API access
- Gmail account with OAuth2 enabled
- FTP server with `facturen/` directory
- PDF.co account (for OCR credits)

### Import Workflows

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Import in this order:
   ```
   1. Flow_A1_Resolve_Invoice_Supplier.json
   2. Flow_A2_Invoice_Classifier.json
   3. Flow_A3_Purchase_Order_Matcher.json
   4. Flow_A4_Ledger_Mapper.json
   5. Flow_Vector_Store_Updater.json
   6. Flow_1_Main_Flow.json   ← last (references all others)
   ```
4. After importing Main Flow, update the `workflowId` values in the **Execute Workflow** nodes to match your imported sub-workflow IDs

---

## 🔑 Environment & Credentials

Set the following credentials in **n8n Settings → Credentials**:

| Credential Name | Type | Used In |
|----------------|------|---------|
| `Gmail Noor` | Gmail OAuth2 | Main Flow |
| `MySQL account Test` | MySQL | Main, A1, A2, A3 |
| `MySQL Prod account` | MySQL | Main Flow (PO queries) |
| `OpenAi account test` | OpenAI API | A1, A2, A3, A4, Vec. Updater |
| `FTP Prod account` | FTP | Main Flow |
| `Mistral Cloud account` | Mistral API | Main Flow (image fallback) |

**PDF.co API Key** — set directly in the HTTP Request node headers:
```
x-api-key: your_pdfco_api_key_here
```

**OpenAI Vector Store ID:**
```
vs_6826ef5e64a0819182f331d03fe628e8
```

---

## 📨 Invoice Processing Flow

```
New Email Arrives (Gmail)
        │
        ▼
Extract Attachments (PDF / JPEG / XML)
        │
        ├─── XML? ──► Parse UBL XML directly
        │
        └─── PDF/JPEG? ──► Upload to PDF.co ──► OCR to text
                               │
                               ▼
                        GPT-4.1 reformat text
        │
        ▼
Log to MySQL (email_logs + processed_files)
        │
        ▼
FTP backup → facturen/errors/{filename}
        │
        ▼
A1: Identify Supplier (OpenAI Assistant + Vector Store)
        │
        ▼
A2: Classify Invoice + Build UBL JSON (OpenAI Assistant)
        │
        ├─── inkoopfactuur (Purchase Invoice)
        │         │
        │         ▼
        │    Get Open POs from ERP (MySQL stored proc)
        │         │
        │         ▼
        │    A3: Match to Purchase Order (OpenAI Assistant)
        │         │
        │         ▼
        │    DB: status = MATCHED or NEW
        │
        ├─── kostenfactuur (Cost Invoice)
        │         │
        │         ▼
        │    A4: Map GL Ledger Codes (OpenAI Assistant)
        │         │
        │         ▼
        │    DB: status = MATCHED or REVIEW
        │
        └─── onbekend (Unknown)
                  │
                  ▼
             DB: status = REJECTED
        │
        ▼
Save JSON result to FTP → facturen/new/{filename}.json
Move invoice file: errors/ → new/
```

---

## 🤖 AI Assistants

| Assistant | ID | Purpose |
|-----------|-----|---------|
| A1 — Supplier Resolver | `asst_mzuHs96x8HxGf8qwp3x9sYsX` | Identifies supplier from OCR text using Vector Store |
| A2 — Invoice Classifier | `asst_PogHknCOyBE2B9Sw1LfvhfDs` | Classifies type + produces structured UBL JSON |
| A3 — PO Matcher | `asst_o7CcGHLadsypmucRXatWXDT1` | Matches invoice lines to open purchase orders |
| A4 — Ledger Mapper | `asst_TqqlQihpuUeeumOzVeLeZkSj` | Assigns general ledger codes to cost invoices |

All assistants use the **OpenAI Assistants API v2** with thread-based memory for context continuity.

---

## 🛡 Error Handling

| Scenario | Strategy |
|---------|----------|
| PDF.co API timeout | `retryOnFail=true`, max 3 retries, 5s wait |
| AI output not parseable | Fallback response (200 OK) — `supplierId=0` |
| No supplier match found | Flow continues — flagged as `onbekend` |
| No PO match | Status = `NEW` — manual AP review queue |
| Duplicate filename | Auto-versioning: `invoice_v2.pdf`, `invoice_v3.pdf` |
| MySQL node error | `onError=continueRegularOutput` |
| Vector Store update error | `onError=continueRegularOutput` on all nodes |

---

## 📊 Invoice Status States

| Status | Table | Meaning |
|--------|-------|---------|
| `PROCESSING` | processed_files | File received, DB record created |
| `OCR Done` | processed_files | Text extracted successfully |
| `SUPPLIER_FOUND` | processed_files | A1 resolved supplier |
| `MATCHED` | aws_invoice_json | Fully matched — no human action needed |
| `NEW` | aws_invoice_json | No PO match — AP team review required |
| `REVIEW` | aws_invoice_json | Ledger mapping needs verification |
| `REJECTED` | aws_invoice_json | Not an invoice or unknown supplier |

---

## 📄 Documentation

Full technical documentation (28 pages) is available in:
```
cocktailmachines_package/documentation/CocktailMachines_n8n_Documentation.pdf
```

Includes: architecture diagrams, node-by-node descriptions, flowcharts for every workflow, database schema, and error handling strategy.

---

## 👤 Author

**Samad Mehboob**  
n8n Automation Developer  
Project: CocktailMachines B.V. Invoice Automation — 2026
Email: **samadmehboob940@gmail.com**

---

*Built with ❤️ using n8n + OpenAI*
