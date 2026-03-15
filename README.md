# 📂 DocuFetch: Enterprise RAG Agent & ETL Pipeline

**DocuFetch** is a high-performance Retrieval-Augmented Generation (RAG) system built on **n8n**. It automates the entire lifecycle of company knowledge—from the moment a document is dropped into **Google Drive** to the moment an employee asks a complex question in **Slack**.

By leveraging **Google Gemini 2.5 Pro** for reasoning and **Pinecone** for vector storage, DocuFetch provides a human-like conversational experience, drawing upon institutional memory.

![Process Flow](docufetch-system-architecture.png)

## 🎥 Demo Video

[Watch the Demo Video](https://www.loom.com/share/8a7bea8d60314eb3ae03c08232b96ad8)

## 📸 Workflow Screenshots

![DocuFetch Workflow Screenshot](Demo_&_Screenshots/DocuFetch_Screenshot.png)

## 🚀 Key Features

* **Automated Knowledge Ingestion (ETL):** A specialized pipeline that polls Google Drive for new files, downloads them, and processes them for the vector database.
* **Advanced Text Chunking:** Uses a Recursive Character Text Splitter with an optimized chunk size and overlap (e.g., 2200 characters with a 200-character overlap) to preserve semantic context across chunks.
* **High-Reasoning LLM:** Powered by Google Gemini 2.5 Pro, enabling the agent to handle nuanced internal queries effectively.
* **Conversational Memory:** Implements a Window Buffer Memory (last **4** interactions) to enable the bot to understand follow-up questions and maintain context.
* **Loop Prevention Logic:** A custom "Ignore Bot" gate ensures the system doesn't trigger itself in Slack, maintaining stability and reducing API costs.

## 🛠️ Tech Stack

| Component           | Technology                           |
| :------------------ | :----------------------------------- |
| **Orchestration**   | n8n                                  |
| **LLM (Reasoning)** | Google Gemini 2.5 Pro                |
| **Vector Database** | Pinecone                             |
| **Embeddings**      | Google text-embedding-004            |
| **Data Sources**    | Google Drive API                     |
| **Communication**   | Slack API                            |
| **Languages**       | JavaScript (Node.js), Python (Flask) |

## 🏗️ Workflow Architecture

The system is divided into two primary loops:

### 1. The Ingestion Loop (ETL)

Every **60 seconds**, the system monitors a specific Google Drive folder.

* **Trigger:** New file detected in "Office Docs".
* **Transform:** Text is extracted and split into optimized segments (see "Advanced Text Chunking").
* **Embed:** Google text-embedding-004 generates high-dimensional vectors for the text.
* **Upsert:** Data is stored in the `documentknowledge` Pinecone index.

### 2. The Retrieval Loop (Query)

When a user sends a message in the `#random` (or designated) Slack channel:

* **Filtering:** The "Ignore Bot" node filters out messages originating from the bot itself to prevent loops.
* **Reasoning:** The Gemini 2.5 Pro Agent analyzes the intent.
* **Retrieval:** The agent calls the `vector_store_retriever` tool to pull relevant facts from Pinecone.
* **Response:** A conversational, brand-consistent answer is sent back to Slack.

## 🧠 Prompt Engineering

The agent is configured with a system prompt that enforces:

* **Internal Awareness:** The bot speaks as a company employee (e.g., "We have..." instead of "The company has...").
* **Source Attribution:** Naturally citing documents (e.g., "According to the Employee Handbook...").
* **Hallucination Prevention:** A key design goal, with ongoing improvements to failure-handling mechanisms.

## 📥 Installation & Setup

1. **Import to n8n:** Download the `DocuFetch_Company_RAG_Agent.json` and import it into your n8n instance (available in the project repository).
2. **Credentials:** Configure the following credentials:

   * Google Drive API
   * Google Gemini API
   * Slack API
   * Pinecone API
3. **Environment Variables:** Update the `folderToWatch` ID and `pineconeIndex` name to match your environment. Also set `pollInterval` if you prefer a different cadence.
4. **Activate:** Toggle the workflow to 'Active'.

## 📈 Impact

* **Automated Maintenance:** Documentation updates are largely automated. Performance can depend on rate limits and chunking strategy, and very large files may benefit from manual review.
* **Reduced Slack Noise:** Employees get instant answers to policy questions without tagging HR/Management.
* **Scalable Knowledge:** Handles thousands of document chunks with typically **sub-second** retrieval times.
