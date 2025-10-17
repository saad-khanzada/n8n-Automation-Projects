#  Gmail Automation Agent

A fully automated **email generation and delivery system** built with **n8n**, integrating **Google Contacts**, **Gmail**, and the **Gemini API** for AI-driven communication.

---

##  Overview

The **Gmail Automation Agent** allows users to send context-aware, AI-generated emails simply through a chat interface.  
It combines automation and natural language intelligence to streamline everyday email tasks.

---

## ⚙️ Core Functionality

1. **Trigger:**  
   - Activated via chat input.  
   - Example command:  
     > “Write an email to Ahmed about the project update.”

2. **Automated Workflow:**
   - Searches the contact in **Google Contacts**.  
   - If the contact **exists** → drafts and sends an AI-written email via **Gmail**.  
   - If the contact **does not exist** → prompts the user to input an email address, then sends the generated message.

3. **AI-Powered Composition:**
   - Uses the **Gemini API** to generate email body, tone, and structure dynamically based on user intent.

---

##  Tech Stack

| Technology | Purpose |
|-------------|----------|
| **n8n** | Workflow orchestration and automation platform |
| **Google Contacts API** | Contact retrieval and validation |
| **Gmail API** | Email drafting and sending |
| **Gemini API** | AI text generation and tone control |

---

## How to Use

1. **Import the Workflow**
   - Open **n8n Dashboard → Workflows → Import from File**
   - Upload the `gmail_automation_agent.json` file from this folder.

2. **Configure Environment**
   - Add the following credentials in n8n:
     - `GEMINI_API_KEY`
     - `GOOGLE_CLIENT_ID`
     - `GOOGLE_CLIENT_SECRET`
   - Adjust any workflow paths or variable names as required.

3. **Run the Automation**
   - Trigger manually or integrate with chat-based inputs.
   - Observe automated message generation and sending.

---

## 📘 Author

Developed by **Saad**,  
 * specialized in Artificial Intelligence, NLP, and Automation Systems.*

> “Transforming everyday tasks into intelligent automations.”

---

## 🪪 License

Licensed under the **MIT License**.  
Feel free to use, adapt, or extend with attribution.
