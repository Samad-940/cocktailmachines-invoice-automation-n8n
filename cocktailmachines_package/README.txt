╔══════════════════════════════════════════════════════════════╗
║         CocktailMachines B.V. — n8n Invoice Automation      ║
║                  Complete Project Package v1.0               ║
╚══════════════════════════════════════════════════════════════╝

PACKAGE CONTENTS:
─────────────────
📁 documentation/
   └── CocktailMachines_n8n_Documentation.pdf
       Complete 28-page technical documentation including:
       • Project overview & architecture
       • System integration map
       • All 6 workflow descriptions (node-by-node)
       • Flowcharts for every workflow
       • Database schema (5 tables)
       • Error handling strategy

📁 flows_json/
   ├── README.txt                          ← Import instructions
   ├── Flow_1_Main_Flow.json               ← Main orchestrator
   ├── Flow_A1_Resolve_Invoice_Supplier.json
   ├── Flow_A2_Invoice_Classifier.json
   ├── Flow_A3_Purchase_Order_Matcher.json
   ├── Flow_A4_Ledger_Mapper.json
   └── Flow_Vector_Store_Updater.json

📁 flow_screenshots/
   ├── README.txt
   ├── Flow_1_Main_Flow_Overview.png
   ├── Flow_A1_Supplier_Resolver.png
   ├── Flow_A2_Invoice_Classifier.png
   ├── Flow_A3_PO_Matcher.png
   ├── Flow_A4_Ledger_Mapper.png
   └── Flow_5_Vector_Store_Updater.png

SYSTEM SUMMARY:
───────────────
Platform   : n8n Workflow Automation
Project    : CocktailMachines Invoice Processing
AI Models  : OpenAI GPT-4.1 + 4x OpenAI Assistants
Databases  : MySQL Test + MySQL Production (ERP)
Integrations: Gmail, FTP, PDF.co API, OpenAI Vector Store
Version    : 1.0 — March 2025

WORKFLOW OVERVIEW:
──────────────────
1. Main Flow     → Email ingestion, OCR, routing
2. A1            → Supplier identification (AI)
3. A2            → Invoice classification + UBL transform (AI)
4. A3            → Purchase order matching (AI)
5. A4            → Ledger/GL code mapping (AI)
6. Vec. Updater  → Supplier knowledge base maintenance

