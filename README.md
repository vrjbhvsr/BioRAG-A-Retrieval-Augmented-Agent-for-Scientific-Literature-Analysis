# BioRAG-A-Retrieval-Augmented-Agent-for-Scientific-Literature-Analysis
Retrieval-Augmented Agent for scientific literature understanding.
---
## Phase 1. Planning and Core Design

**Key Steps:**

* Problem Definition & Goal Setting
* Success Metrics Definition (KPIs)
* Data Availability & Feasibility Assessment
* Initial Architecture & Tool Selection

### 1. Problem Definition & Goal Setting
### Problem Statement

The rapid growth of scientific litratures in any domain including BioMedical has made it difficult for the researchers to identifying key points from the paper efficiently. Researchers often struggles to extract meaningful findings, evidence and information for decision-making due to:

* Information Overload
* Unclear Paper Structure
* Dense Languages
* Lack of Context
* Time constraints
* Overly Detailed methods

The traditional models and methods require manual effort to interprete the generated unstrucutred results. The growth of LLM is able to provide useful information the way user want, they still offeres problems like hallucinations, less reliability, lack of tracability. It's necessary to develop more robust system that assist researchers to interprete the paper and retrieve the relevent information that saves manual efforts and time.

---
### Core Objective

To develop and validate a high-quality, Retrieval-Augmented Generation (RAG) chatbot agent capable of performing deep, structured analysis on the initial scientific research paper (PDF). The primary goal is to provide accurate, comprehensive, and traceable answers to 8 core user queries (e.g., experimental setup, key claims), delivered in a clear and consistent format suitable for a human researcher.

The 8 core data points are:

* Get the essense of the paper
* Get the **Why** of the paper
* Get the **What** from the paper
* Get the **Experimental setup**
* Get the **Materials** used in experments
* Get the **Methods** followed to achieve results
* Interprete the **Tables** and **Results**
* Find the **Claims**

---

### Success Metrics Definition (KPIs)

A primary user for this BioRAG is a researchers, defining a KPIs based on their perspective is necessary.

There will be five major **Key Performance Indicators** grouped by what they measures. There are three groups for RAG:

* The Retriever
* The Generator
* System Usability and Structure

#### The Retriever (Retrieval Quality)
These metrics ensure the system is finding the most relevent sections of the paper to answer the queries.

* **Context Relevence:** This Metric will measure the percentage of the retireved chunks that are highly relevent and useful for generating fnial answer.

* **Context Recall:** This metric measures whether the retrieved context contains all the information to completely answer the question. Some question like experimental setup spans multiple section in the paper, this metric is very useful in this scenarios.

#### The Generator (Trustworthiness and Factual accuracy)
This metric ensure that the system is not hallucinating and the output is factually correct.

* **Faithfulness:** This metric measure the generated answer is reliabe and supported by the retrieved chunk from the paper.

* **Factual accuracy:** Measures whether the final output is factually correct against a "Gold Standard" human-verified answer for the 8 core queries.

#### System Usability and Structure

These metrics measures the user experiance.

* **Latency:** The total time taken from the user submitting one of the 8 queries to receiving the final answer.

### Data Availability & Feasibility Assessment

This document summarizes the pilot test results for core components of the **BioRAG** system, identifies associated risks and constraints, and records the architectural commitments for the next development phases.

---

#### 1. Loader / Parser

#### **Pilot Test Result**
- `pypdf` and `PyMuPDF` produced unreliable structural parsing; extracted text appeared scrambled or disordered.  
- LangChain’s **UnstructuredLoader** offered the best structural output and successfully identified major text categories.  
- Drawbacks: Slow performance (~1 min/document) and occasional minor category misclassification.

#### **Risks & Constraints Identified**
- **High latency** reduces throughput for large datasets.  
- **Mis-tagging** affects segmentation and downstream retrieval accuracy.

#### **Architectural Commitment**
- Adopt **UnstructuredLoader** as the primary parser due to its superior structural accuracy.  
- Implement a **Validation & Correction Layer (Phase 2)** to handle category errors.  
- Accept slower parsing speed in exchange for structurally clean, reliable output.

---

#### 2. Output Parser

#### **Pilot Test Result**
- `PydanticOutputParser` frequently failed with open-source LLMs (e.g., Qwen), causing JSON schema validation errors.  
- LLMs struggled to produce rigid, schema-aligned outputs consistently in a single generation pass.

#### **Risks & Constraints Identified**
- Breaks **MAS (Multi-Agent System) readiness**, which requires structured message passing.  
- High likelihood of runtime failures during schema validation.

#### **Architectural Commitment**
- Implement a **Robust Multi-Step Parsing Strategy (Phase 3)** that includes:  
  - Retry loops  
  - Error feedback prompts  
  - Optional **LLM-as-a-Judge** or self-correction patterns  
- Ensure the final output fully complies with the Pydantic schema before inter-agent communication.

---

#### 3. Embedding Model

### **Pilot Test Result**
- General-purpose embeddings (e.g., Qwen2.5-8B) performed poorly on biomedical and materials science terminology.  
- Semantic coherence was weak, leading to suboptimal retrieval results.

#### **Risks & Constraints Identified**
- **Low context relevance** and poor domain specificity.  
- Critical scientific terminology is not well represented by generalist embedding models.

#### **Architectural Commitment**
- Transition to **scientific domain-specific or instruction-tuned embedding models**, such as:  
  - **e5-base-instruct**  
  - **SciBERT embeddings**  
- Improve semantic relevance and retrieval precision within scientific corpora.

---

#### 4. Vector Store

#### **Pilot Test Result**
- FAISS demonstrated fast and efficient local performance.  
- However, it lacks advanced features needed for scientific retrieval workflows.

#### **Risks & Constraints Identified**
- No **metadata filtering** support natively.  
- No **hybrid search** (vector + keyword).  
- Limits the ability to isolate context (e.g., retrieving only “Materials” within “Methods”).

#### **Architectural Commitment**
- Upgrade to a vector store that supports both **metadata filtering** and **hybrid retrieval**, such as:  
  - **Weaviate**  
  - **Qdrant**  
  - **Milvus**  
  - **Elasticsearch kNN**


### Initial Architecture and Tool Selection



The core goal of Step 1.4 is to finalize the RAG stack and retrieval strategy, specifically designed to overcome the structural parsing challenges identified in the feasibility assessment (1.3) and ensure compliance with the **Structured Output Reliability (100%)** KPI.

### 1. The RAG Stack (Tool Commitments)

Based on the pilot failures of simple tools (pypdf, PyMuPDF) and the unreliability of generalist open-source models, we mandate the selection of specialized components:

#### A. Loader / Parser
* **Selected Tool:** **LangChain's UnstructuredLoader** (for element categorization) integrated with **Camelot** or **pdfplumber** (for dedicated table extraction).
* **Rationale:** The UnstructuredLoader was the best performer for structural integrity, providing essential element tags (`header`, `table`, `text`). The trade-off is high latency ($\approx$ 1 minute/document), which is deemed acceptable for the ingestion pipeline's quality assurance. Dedicated table parsing is required to extract structured data for the 8 core queries.
