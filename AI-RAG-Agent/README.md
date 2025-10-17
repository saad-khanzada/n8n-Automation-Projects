# 🧠 RAG Agent Automation

A Retrieval-Augmented Generation (RAG) automation workflow built in **n8n**, integrating **Gemini API** for contextual reasoning and **Supabase** for vector data retrieval.

---

## ⚙️ Setup Instructions

1. **Import Workflow**
   - Open your n8n dashboard → *Workflows → Import from File*  
   - Select `rag_workflow.json` from this folder.

2. **Configure Environment**
   - Add your API keys:
     - `GEMINI_API_KEY`
     - `SUPABASE_URL`
     - `SUPABASE_KEY`
   - Adjust database table names or paths if required.

3. **Run Workflow**
   - Test interactively through n8n chat UI  
   - Or schedule via automation triggers.

---

## 🧩 Tech Stack

| Technology  | Purpose |
|--------------|----------|
| n8n | Visual automation platform orchestrating RAG flow |
| Gemini API | AI reasoning, contextual generation |
| Supabase | Vector database for document retrieval |
| Google APIs | Optional for Gmail/Drive integration |

---

## 📘 Author

Developed by **Saad**, t specializing in **AI, NLP, and Automation**.  
*"Integrating intelligence into automation — one workflow at a time."*
