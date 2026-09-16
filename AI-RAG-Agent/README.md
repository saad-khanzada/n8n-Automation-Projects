# RAG Document Q&A Agent with n8n, Supabase and Gemini

A working document-Q&A prototype built with **n8n**, **Google Gemini**, **Supabase pgvector**, and **Google Drive**.

The workflow ingests a PDF from Google Drive, converts the document into Gemini embeddings, stores the resulting chunks in Supabase, and lets an AI agent retrieve relevant passages before answering a user's question.

## Project status

- Working prototype
- Recovered and revalidated in September 2026
- No permanent public deployment
- No formal retrieval benchmark or overall accuracy score

## What the workflow does

### Ingestion path

```text
Google Drive PDF
      ↓
Default Data Loader
      ↓
Gemini Embeddings
      ↓
Supabase Vector Store
```

### Retrieval path

```text
User Question
      ↓
RAG Agent
      ↓
Query Embedding
      ↓
Supabase Vector Search
      ↓
Relevant Document Chunks
      ↓
Gemini Chat Model
      ↓
Grounded Answer
```

The retrieval tool is currently configured with `topK: 2`.

## Document used for validation

The recovered workflow was validated against the OECD report:

**Artificial Intelligence and the Changing Demand for Skills in the Labour Market**  
OECD Artificial Intelligence Papers, No. 14, April 2024

During recovery, the old workflow was found to contain a system prompt for a different OECD report. The prompt was corrected to match the actual document used by the workflow.

## Recovery and migration work

The original local n8n/Docker environment was no longer available, so the workflow was recovered from its exported JSON and restored in n8n Cloud.

The recovery included:

- Reconnecting the original Google Drive document source
- Resuming the original Supabase project
- Verifying and backing up **442 historical vector records** before modifying the active vector store
- Diagnosing a failed ingestion caused by unusable embedding output
- Moving the active embedding configuration from **768 dimensions to 3072 dimensions**
- Reprocessing the original **55-page PDF**
- Storing **212 fresh document chunks** in the active `documents` table
- Reconnecting Gemini API access
- Restoring the retrieval path between the AI agent, Gemini embeddings, and Supabase
- Correcting the AI agent's system prompt to match the actual OECD report

## Verified test example

A controlled question asked the system to compare the share of vacancies in high AI-exposure occupations that demand at least one management skill with those that demand at least one business processes skill.

The workflow returned:

- **Management skill:** 72%
- **Business processes skill:** 67%
- **Difference:** 5 percentage points

Both values matched the source report, and the n8n execution trace showed the Supabase retrieval and Gemini embedding nodes running during the request.

This is a validation example, not an overall accuracy measurement.

## Technologies

| Technology | Role |
|---|---|
| n8n | Workflow orchestration and agent flow |
| Google Drive | Source document download |
| Google Gemini | Embeddings and chat generation |
| Supabase | Vector storage and similarity retrieval |
| pgvector | 3072-dimensional vector column and cosine-distance search |

## Requirements

A separate `requirements.txt` or dependency file is not needed because this is an n8n workflow rather than a Python application.

You need:

- An n8n instance, either Cloud or self-hosted
- A Google Drive account and OAuth credential in n8n
- A Google Gemini API credential in n8n
- A Supabase project
- pgvector enabled in Supabase
- A PDF stored in Google Drive

## Supabase setup

For a fresh setup, run the following in the Supabase SQL editor.

> If you already have a `documents` table with data, back it up before changing vector dimensions.

```sql
create extension if not exists vector;

create table if not exists public.documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(3072)
);

create or replace function public.match_documents (
  query_embedding vector(3072),
  match_count int default null,
  filter jsonb default '{}'
)
returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
#variable_conflict use_column
begin
  return query
  select
    id,
    content,
    metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from public.documents
  where metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

## Import and configuration

### 1. Import the workflow

Download [`AI-RAG-Agent.json`](./AI-RAG-Agent.json), then in n8n import it from a file.

The public JSON is intentionally sanitized. It does **not** contain my credentials, account-specific credential IDs, private Google Drive file ID, or n8n instance metadata.

### 2. Connect your credentials

After importing, assign your own credentials to these nodes:

- `Download file` → Google Drive OAuth credential
- `Supabase Vector Store` → Supabase credential
- `Supabase Vector Store1` → Supabase credential
- `Embeddings Google Gemini` → Gemini API credential
- `Google Gemini Chat Model` → Gemini API credential

Never place API keys directly inside the workflow JSON before committing it to a public repository.

### 3. Choose your Google Drive PDF

Open the `Download file` node and select your own PDF from Google Drive.

The public workflow contains this placeholder instead of my original file ID:

```text
YOUR_GOOGLE_DRIVE_FILE_ID
```

You can replace it through the n8n node interface rather than editing the JSON manually.

### 4. Check the embedding model

The workflow uses:

```text
models/gemini-embedding-001
```

The current setup stores **3072-dimensional vectors**. The Supabase vector dimension must match the embedding output.

Use the same embedding model for both document ingestion and query retrieval. Do not mix embeddings generated by different models in the same active vector store.

### 5. Check the chat model

The exported workflow currently references a Gemini Flash chat model. Model availability can differ by Gemini API project and over time.

If the imported chat node reports that the configured model is unavailable, select a currently available Gemini Flash model from the node. Changing the chat-generation model does not require rebuilding the vector store, but changing the embedding model does.

### 6. Run ingestion

Run the manual ingestion branch first:

```text
Manual Trigger
→ Download file
→ Supabase Vector Store
```

The data loader and Gemini embedding nodes feed the vector store during this execution.

After the run finishes, confirm that rows were inserted into `public.documents`.

### 7. Test retrieval

Open the n8n chat interface and ask a question that can be answered from your document.

During a successful RAG request, you should see the retrieval path execute through the AI agent, Supabase Vector Store, Gemini embeddings, and Gemini chat model.

## Security notes

- The public workflow file is sanitized and contains no saved n8n credential references.
- Add credentials inside your own n8n instance after import.
- Do not commit API keys, OAuth tokens, Supabase service-role keys, passwords, or private connection strings.
- If a real secret was ever committed to a public repository, rotate or revoke it. Removing it from the newest file does not remove it from Git history.

## Limitations

- No permanent public deployment is included
- The current validation uses controlled factual checks, not a full RAG evaluation benchmark
- No measured retrieval precision, recall, or overall answer accuracy is claimed
- Production monitoring, access control, scaling, and security hardening are outside the scope of this prototype

## Possible next improvements

- Add source citations or retrieved passage previews to answers
- Evaluate retrieval quality with a larger question set
- Add reranking or hybrid retrieval
- Support multiple documents and document-level filtering
- Deploy the workflow on a stable always-on environment

## License

This repository is licensed under the MIT License.
