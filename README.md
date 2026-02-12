# Bedrock Agent - Intelligent FAQ Assistant

An intelligent FAQ assistant built with LangGraph and AWS Bedrock AgentCore, designed to answer customer queries using advanced RAG (Retrieval-Augmented Generation) techniques with semantic search over a knowledge base.

## 🎯 Overview

This project demonstrates a production-ready FAQ chatbot that:
- **Searches** a knowledge base using FAISS vector similarity
- **Retrieves** relevant FAQ entries using semantic embeddings
- **Generates** accurate responses using LLM-powered reasoning
- **Deploys** seamlessly to AWS Bedrock AgentCore runtime

The agent is built around a Lauki Phones carrier knowledge base containing 76 comprehensive Q&A entries covering topics like billing, roaming, device compatibility, network issues, and more.

## 🏗️ Architecture

```
┌─────────────────┐
│  User Query     │
└────────┬────────┘
         │
         ▼
┌─────────────────────────────┐
│   LangGraph Agent           │
│   (ChatGroq LLM)            │
└────────┬────────────────────┘
         │
         ├──► search_faq (k=3)
         ├──► search_detailed_faq (k=5)
         └──► reformulate_query
                  │
                  ▼
         ┌────────────────────┐
         │  FAISS Vector DB   │
         │  (HuggingFace      │
         │   Embeddings)      │
         └────────────────────┘
                  │
                  ▼
         ┌────────────────────┐
         │   CSV Knowledge    │
         │   Base (76 FAQs)   │
         └────────────────────┘
```

### Core Components

1. **LangGraph Agent** - Orchestrates tool selection and reasoning
2. **FAISS Vector Store** - Enables fast semantic similarity search
3. **HuggingFace Embeddings** - `sentence-transformers/all-MiniLM-L6-v2` for sentence embeddings
4. **ChatGroq LLM** - `openai/gpt-oss-20b` model for intelligent response generation
5. **AWS Bedrock AgentCore** - Serverless deployment runtime with API endpoints

## 🛠️ Technologies

| Technology | Purpose |
|------------|---------|
| **LangGraph** | Agent workflow orchestration |
| **LangChain** | LLM integration framework |
| **FAISS** | Vector similarity search |
| **AWS Bedrock AgentCore** | Serverless agent deployment |
| **HuggingFace Transformers** | Sentence embeddings |
| **Groq** | High-performance LLM inference |
| **Python 3.13+** | Core language |

## 📦 Installation

### Prerequisites

- Python 3.13 or higher
- AWS Account (for Bedrock AgentCore deployment)
- Groq API Key (get from [console.groq.com](https://console.groq.com))

###Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/Ayush95697/Bedrock_agent.git
   cd Bedrock_agent
   ```

2. **Install dependencies**
   ```bash
   pip install uv
   uv sync
   ```

3. **Configure environment variables**
   
   Create a `.env` file:
   ```env
   OPENAI_API_KEY=your_groq_api_key_here
   ```

4. **Run locally**
   ```bash
   python 00_langraph_agent.py
   ```

## 🚀 Usage

### Local Execution

The standalone script demonstrates the agent in action:

```python
from 00_langraph_agent import agent

# Query the agent
result = agent.invoke({
    "messages": [("human", "How do I activate international roaming?")]
})
print(result['messages'][-1].content)
```

**Example Output:**
```
To activate international roaming:
1. Purchase a region-specific roaming pack
2. Enable roaming in your device settings
3. The system will push roaming credentials to the visited network within minutes

Available packs cover APAC, EU, Middle East, and Americas with specific data caps and validity periods.
```

### AWS Bedrock AgentCore Deployment

The `agentcore_runtime.py` file provides a serverless entrypoint for AWS deployment:

```python
# Deployed endpoint accepts payload
{
  "prompt": "What plans do Lauki Phones offer?"
}

# Returns structured response
{
  "result": "Lauki Phones offers prepaid, postpaid, family, enterprise, 
             and data-only plans..."
}
```

**Deploy to AWS:**
```bash
# Configure AWS credentials
aws configure

# Deploy via AgentCore CLI
bedrock-agentcore deploy
```

## 🔧 Agent Tools

The agent has access to three specialized tools:

### 1. `search_faq`
- **Purpose:** Quick search for relevant FAQ entries
- **Returns:** Top 3 most relevant results
- **Use Case:** Fast answers to common questions

### 2. `search_detailed_faq`
- **Purpose:** Comprehensive search for complex queries
- **Returns:** Top 5+ configurable results
- **Use Case:** Multi-faceted questions requiring broader context

### 3. `reformulate_query`
- **Purpose:** Search focused on specific aspects (pricing, activation, troubleshooting)
- **Returns:** Targeted results for a specific query dimension
- **Use Case:** When initial search lacks focus

### Agent Strategy

The system prompt guides the agent to:
1. Start with `search_faq` for initial retrieval
2. Use `search_detailed_faq` if more context is needed
3. Employ `reformulate_query` to explore different query angles
4. Synthesize information from multiple tool calls
5. Provide clear, concise answers or state when information is unavailable

## 📊 Knowledge Base

The `lauki_qna.csv` contains 76 structured Q&A entries covering:

- **Account Management:** Activation, KYC, billing, payments
- **Plans & Services:** Prepaid, postpaid, family plans, roaming
- **Technical Support:** Network issues, VoLTE, WiFi calling, device compatibility
- **Advanced Features:** Number porting, conference calling, enterprise APIs
- **Compliance & Security:** Data protection, MFA, spam blocking

Each entry is chunked and embedded for efficient semantic retrieval.

## 🔐 Security & Configuration

### Environment Variables

| Variable | Description | Required |
|----------|-------------|----------|
| `OPENAI_API_KEY` | Groq API key for LLM access | Yes |

### AWS Configuration

See `.bedrock_agentcore.yaml` for deployment configuration:
- **Region:** `ap-south-1`
- **Account:** Configured per deployment
- **Runtime:** Containerized Python with HTTP protocol
- **Network:** Public endpoint with observability enabled

## 📈 Performance

- **Embedding Model:** Lightweight `all-MiniLM-L6-v2` (80MB)
- **Vector Search:** Sub-millisecond FAISS lookups
- **Chunk Size:** 500 characters (optimized for FAQ entries)
- **LLM Latency:** ~1-2s on Groq infrastructure
- **Cold Start:** ~5-10s on AWS Bedrock AgentCore

## 🧪 Testing

Test the agent with sample queries:

```python
test_queries = [
    "How do I check my data balance?",
    "What should I do if my SIM is lost?",
    "Does Lauki Phones support 5G?",
    "How do I enable WiFi calling?",
    "Are there corporate discounts available?"
]

for query in test_queries:
    result = agent.invoke({"messages": [("human", query)]})
    print(f"Q: {query}")
    print(f"A: {result['messages'][-1].content}\n")
```

## 🤝 Contributing

Contributions are welcome! To extend the knowledge base:

1. Add new Q&A entries to `lauki_qna.csv`
2. Follow the format: `question,answer`
3. Rebuild the FAISS index by running the agent
4. Test with relevant queries

## 📝 License

This project is licensed under the MIT License - see LICENSE file for details.

## 🙏 Acknowledgments

- **LangChain** for the agent framework
- **HuggingFace** for open-source embeddings
- **FAISS** by Meta AI for vector search
- **AWS Bedrock** for serverless runtime
- **Groq** for fast LLM inference

## 📞 Contact

**Author:** Ayush  
**GitHub:** [@Ayush95697](https://github.com/Ayush95697)  
**Project:** [Bedrock_agent](https://github.com/Ayush95697/Bedrock_agent)

---

**Built with ❤️ using LangGraph and AWS Bedrock AgentCore**
