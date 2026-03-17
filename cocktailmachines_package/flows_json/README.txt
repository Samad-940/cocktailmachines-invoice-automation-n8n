==============================================
  CocktailMachines n8n Flows — JSON Files
==============================================

How to import into n8n:
-----------------------
1. Open your n8n instance
2. Click "+" to create new workflow
3. Click the three dots menu (top right) → "Import from File"
4. Select the desired .json file
5. Click "Import"

Files in this folder:
---------------------
Flow_1_Main_Flow.json
  → Main orchestrator flow
  → Trigger: Gmail (every minute)
  → Handles: Email ingestion, OCR, FTP, routing to sub-flows

Flow_A1_Resolve_Invoice_Supplier.json
  → OpenAI Assistant: asst_mzuHs96x8HxGf8qwp3x9sYsX
  → Purpose: Identify supplier from invoice OCR text

Flow_A2_Invoice_Classifier.json
  → OpenAI Assistant: asst_PogHknCOyBE2B9Sw1LfvhfDs
  → Purpose: Classify invoice type + transform to UBL JSON

Flow_A3_Purchase_Order_Matcher.json
  → OpenAI Assistant: asst_o7CcGHLadsypmucRXatWXDT1
  → Purpose: Match invoice to open purchase orders

Flow_A4_Ledger_Mapper.json
  → OpenAI Assistant: asst_TqqlQihpuUeeumOzVeLeZkSj
  → Purpose: Assign GL ledger codes to cost invoices

Flow_Vector_Store_Updater.json
  → Trigger: Manual
  → Purpose: Maintain OpenAI Vector Store supplier knowledge base
  → Vector Store ID: vs_6826ef5e64a0819182f331d03fe628e8

Required Credentials (set in n8n Settings → Credentials):
----------------------------------------------------------
- Gmail OAuth2 (name: Gmail Noor)
- MySQL Test DB (name: MySQL account Test)
- MySQL Prod DB (name: MySQL Prod account)
- OpenAI API (name: OpenAi account test)
- FTP Prod (name: FTP Prod account)
- Mistral Cloud (name: Mistral Cloud account)
- PDF.co API key: set in HTTP Request node headers
