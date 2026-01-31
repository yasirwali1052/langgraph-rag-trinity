# RAG Trinity

<div align="center">
  <img src="assets/rag-agent-workflow.png" alt="RAG Agent Workflow" width="800"/>
</div>

## Overview
A smart RAG agent that finds accurate answers by searching documents and the web. It checks itself at every step to make sure the information is relevant and the answers are truthful, not made up. Uses three quality control methods (Adaptive, Corrective, and Self-RAG) to give you reliable responses you can trust.

---

## Architecture

### The Trinity Approach

**Adaptive RAG** intelligently routes questions to the right information source. Technical queries go to the knowledge base while current events trigger web search.

**Corrective RAG** fixes retrieval mistakes automatically. When documents are irrelevant, the system searches the web instead of generating poor answers.

**Self RAG** validates every answer through multiple checks. It verifies facts are grounded in sources and confirms responses actually address the question.

---

## How It Works

### Phase 1: Intelligent Routing

Every question enters through a smart router that analyzes the topic. Questions about LLM agents, prompt engineering, or adversarial attacks are recognized as specialized topics covered in the knowledge base. The router sends these to the vector store for retrieval. Everything else, especially questions needing current information, gets routed to web search immediately.

### Phase 2: Retrieval and Grading

**Vector Store Path**: The system converts the question into a vector and searches the database for the two most similar document chunks. Each retrieved chunk gets scored for relevance. If documents contain keywords and concepts related to the question, they pass. Irrelevant documents get filtered out.

**Web Search Path**: Tavily searches the internet and returns the top three results. Content from these pages gets formatted as documents and added to the context.

### Phase 3: Quality Control Loop

**Document Assessment**: After retrieval, the system checks if any relevant documents were found. If everything got filtered out as irrelevant, it activates web search as a backup rather than trying to answer from nothing.

**Answer Generation**: The language model receives the question and all relevant documents. It generates a concise answer using only the provided context, instructed to say when it does not know rather than guess.

**Hallucination Check**: The system examines whether every claim in the answer appears in the source documents. Unsupported statements fail this test.

**Relevance Check**: Even factually correct answers get verified for usefulness. Does this actually answer what the user asked?

### Phase 4: Adaptive Response

**Success**: If the answer passes both checks, the user receives it.

**Hallucination Detected**: The system regenerates the answer, attempting to stick closer to the source material.

**Irrelevant Answer**: Web search activates to find better information, then generation runs again.

---

## Technical Components

### Knowledge Base
Three technical blog posts about LLM technology get loaded, split into 512-token chunks, converted to vector embeddings, and stored in ChromaDB. This creates a searchable index of 70+ document chunks.

### Language Models
FastEmbed converts text to vectors for similarity search. Llama 3.1 through Groq handles all reasoning tasks including routing, grading, and generation.

### Workflow Engine
LangGraph orchestrates the entire process as a state machine. Information flows through nodes for retrieval, grading, generation, and validation. Conditional edges create loops for retries and corrections.

---

## What Makes It Reliable

Traditional RAG systems retrieve documents and generate answers blindly. This system adds five quality gates:

1. Route questions to the best information source
2. Grade retrieved documents for relevance
3. Trigger corrective web search when needed
4. Verify answers are grounded in sources
5. Confirm answers address the question

Failed checks do not produce bad output. Instead, they trigger corrective actions: filter documents, search the web, or regenerate answers. This creates a self-healing system that catches and fixes its own mistakes.

---

## Example Workflows

**"What is prompt engineering?"**
Router identifies this as a knowledge base topic. Vector store retrieves relevant chunks from the prompt engineering blog post. Documents pass relevance grading. Answer gets generated, passes hallucination and relevance checks, and returns to the user.

**"Who are the Bears drafting first?"**
Router recognizes this needs current information. Web search activates immediately. Tavily returns recent NFL draft news. Answer gets generated from web results, passes validation, and returns to the user.

**"What are the types of agent memory?"**
Router sends to vector store. Retrieved documents get mixed relevance scores. Some chunks pass, others fail. The system notices incomplete retrieval and activates corrective web search. Combined information from both sources generates a comprehensive answer that passes all checks.

---

## Setup Requirements

You need two API keys to run this system:

**Groq API**: Free tier available at console.groq.com for accessing Llama 3.1

**Tavily API**: Free tier available at tavily.com for web search functionality

The implementation uses Google Colab or any Python environment with the following libraries: langchain, langchain-community, langchain-groq, chromadb, langgraph, tavily-python, and fastembed.

---

## Performance Characteristics

The system typically completes queries in 2 to 5 seconds depending on the path. Vector store retrieval takes under 1 second. Web search adds 1 to 2 seconds. Each LLM call for routing, grading, or generation takes 0.3 to 1 second. Multiple quality checks create slight overhead but prevent costly mistakes like hallucinations or irrelevant answers.

The corrective mechanisms mean the system may make multiple attempts before returning an answer, but each attempt improves quality. Users get reliable information rather than fast garbage.

---

## Why This Matters

Most RAG systems fail silently. They retrieve irrelevant documents, hallucinate facts, or answer different questions than asked. Users cannot tell good answers from bad ones without manual verification.

This implementation makes quality control automatic. Every answer gets validated before reaching the user. Mistakes trigger corrections rather than propagating through to output. The result is a RAG system you can actually trust for important questions.