
#  AI RAG Agent

An intelligent **Retrieval-Augmented Generation (RAG) Agent** built using **n8n**, **Supabase**, and the **Gemini API**, designed to deliver context-aware, fact-grounded responses from document-based knowledge.

---

## Core Functionality

- ** PDF Ingestion & Chunking:**  
  The workflow processes PDF documents, converts them into embeddings, and stores them in a **Supabase vector database** for fast contextual retrieval.

- ** Chat-Based Querying:**  
  When a user sends a query, the system retrieves the most relevant text chunks from the Supabase database.

- ** Intelligent Response Generation:**  
  Leveraging **Gemini LLM**, the agent synthesizes retrieved data into accurate, context-rich, and coherent answers.

---

##  Use Case Example

This project demonstrates how to enable document-grounded Q&A using the **OECD 2023 Report**:  
> “Emerging Trends in AI Skill Demand Across 14 OECD Countries”

Users can ask questions like:
- “Which countries report the highest AI skill demand growth?”
- “What are the key emerging skills in AI-related jobs?”

The agent responds with **evidence-based summaries** derived directly from the document.

---

##  Setup Instructions

### 1️ Import Workflow
- Open your **n8n dashboard**
- Navigate to → *Workflows → Import from File*
- Select the JSON file from this folder (`rag_workflow.json`)

### 2️ Configure Environment
- Add and configure the following API keys:
  - `GEMINI_API_KEY`
  - `SUPABASE_URL`
  - `SUPABASE_KEY`
- Adjust paths or database references if necessary.

### 3️ Run the Workflow
- Test interactively through the n8n chat interface.  
- Optionally, schedule triggers for automated execution.

---

##  Tech Summary

| Technology | Purpose |
|-------------|----------|
| **n8n** | Visual automation platform orchestrating workflows |
| **Gemini API** | AI reasoning, text generation, and contextual chat responses |
| **Supabase** | Vector database and data storage layer |
| **Vector Embeddings** | Enable semantic document retrieval for RAG |

---

##  Future Enhancements

- Integration with **Google Drive** for dynamic document uploads.  
- **Multi-document context retrieval** using hybrid search.  
- **Chat UI Interface** for end-user interactions.

---

##  About the Developer

Developed by **Saad**,  specializing in **Artificial Intelligence**, **NLP**, and **Automation Systems**.  
Focused on creating scalable, real-world AI-driven automations through open-source experimentation.

> “Turning static data into intelligent, conversational knowledge systems.”

---

##  License

This project is licensed under the **MIT License**.  
Feel free to use, modify, or extend this workflow with proper attribution.
