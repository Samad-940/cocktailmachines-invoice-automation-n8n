==============================================
  CocktailMachines n8n — Flow Screenshots
==============================================

Screenshots taken directly from n8n canvas:

Flow_1_Main_Flow_Overview.png
  → Full overview of the main orchestrator flow
  → Shows all phases: email ingestion → OCR → routing → sub-flows

Flow_A1_Supplier_Resolver.png
  → A1: Resolve Invoice Supplier sub-workflow
  → Trigger → AI Assistant → Parse → DB Update → Response

Flow_A2_Invoice_Classifier.png
  → A2: Invoice Classifier & Transformer sub-workflow
  → Trigger → AI Classifier → SQL Lookups → Switch → Response

Flow_A3_PO_Matcher.png
  → A3: Purchase Order Matcher sub-workflow
  → Trigger → Parse Orders → AI Matcher → Parse Output → Response

Flow_A4_Ledger_Mapper.png
  → A4: Ledger Mapper sub-workflow
  → Trigger → Parse JSON → AI Matcher → Validate → Response

Flow_5_Vector_Store_Updater.png
  → Vector Store Updater (background maintenance flow)
  → Handles: Delete old chunks + Create/Update supplier profiles
