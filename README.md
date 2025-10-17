# 🤖 n8n Automation Projects by Saad

A curated collection of **AI-driven automation workflows** built with [n8n](https://n8n.io/), leveraging **Gemini API**, **Supabase**, and various Google integrations.  
These projects demonstrate how intelligent automations can simplify communication, enhance productivity, and integrate AI reasoning into real-world workflows.

---

## 🧩 Project Overview

### 1. 🧠 AI RAG Agent
**Description:**  
An intelligent Retrieval-Augmented Generation (RAG) agent designed using **n8n**, **Supabase**, and the **Gemini API**.  

**Core Functionality:**
- Ingests and chunks PDF data into a **Supabase vector database**.  
- Uses a **chat trigger**: when a user asks a question, it retrieves relevant sections from stored data.  
- Generates context-aware, fact-based answers using the **Gemini LLM**.  

**Use Case Example:**  
This implementation uses the *OECD 2023 Report: “Emerging Trends in AI Skill Demand Across 14 OECD Countries”* to answer policy-related or educational trend questions with precision.  

**Tech Stack:**  
`n8n` · `Supabase` · `Gemini API` · `Vector Embeddings`  

📁 [AI RAG Agent →](./AI-RAG-Agent/README.md)

---

### 2. 📧 Gmail Automation Agent
**Description:**  
A fully automated **email generation and sending workflow** that integrates **Google Contacts**, **Gmail**, and the **Gemini API** inside n8n.  

**Core Functionality:**
- Triggered via chat input.  
- When a user says: “Write an email to [Name]”, the system:
  1. Searches Google Contacts for the name.  
  2. If found → drafts and sends an AI-written email via Gmail.  
  3. If not found → prompts the user to enter an address, then sends the generated message automatically.  
- Utilizes the **Gemini API** for tone, structure, and content generation.  

**Tech Stack:**  
`n8n` · `Google Contacts API` · `Gmail API` · `Gemini API`  

📁 [Gmail Automation Agent →](./Gmail-Automation-Agent/README.md)

---

## ⚙️ Setup & Usage

1. **Clone this Repository**
   ```bash
   git clone https://github.com/saad-khanzada/n8n-Automation-Projects.git
   cd n8n-Automation-Projects






### 🧩 2. Import a Workflow
1. Open your **n8n Dashboard** → navigate to **Workflows** → click **Import from File**.  
2. Select the appropriate **`.json`** file from the desired project folder.

---

### 🔧 3. Configure Environment
1. Add required **API Keys**:
   - **Gemini API**
   - **Supabase API**
   - **Google APIs** (Gmail, Contacts, etc.)
2. Update:
   - **Paths** or **database references**, if required.
   - Any environment variables or secrets according to your local setup.

---

### 🚀 4. Run the Workflow
1. Test the automation **interactively via chat trigger**, or  
2. Schedule it to run automatically using **n8n’s built-in triggers**.

---

## 🧠 Tech Summary

| **Technology** | **Purpose** |
|-----------------|-------------|
| **n8n** | Visual automation platform orchestrating workflows |
| **Gemini API** | AI reasoning, text generation, and contextual chat responses |
| **Supabase** | Vector database and data storage layer |
| **Google APIs** | Gmail & Contacts automation and message delivery |

---

## 🚀 Future Roadmap

| **Planned Workflow** | **Description** |
|------------------------|-----------------|
| **WhatsApp Automation Agent** | Send or schedule WhatsApp messages using AI-written content |
| **YouTube Caption Generator** | Automate caption and description creation using the Gemini API |
| **Google Sheets Data Sync** | Integrate Gemini for intelligent spreadsheet updates |

---

## 📘 About the Author

Developed by **Saad**, a **BSCS student** specializing in **Artificial Intelligence, NLP, and Automation Systems**.  
Focused on building **scalable, real-world AI solutions** through open-source experimentation and automation design.

> *“Turning repetitive processes into intelligent automations — one workflow at a time.”*

---

## 🪪 License

This repository is licensed under the **MIT License**.  
You are free to **use, modify, and distribute** these workflows with proper attribution.
