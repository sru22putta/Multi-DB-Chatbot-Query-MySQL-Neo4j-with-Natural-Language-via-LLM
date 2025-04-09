# Multi-DB-Chatbot-Query-MySQL-Neo4j-with-Natural-Language-via-LLM
An LLM-powered chatbot that lets users query real-time sensor data (temperature, occupancy) from MySQL and explore data center layouts via Neo4j. It uses GPT-4o and LangChain to route queries, generate SQL/Cypher, and return natural language answers. Built with Streamlit for an intuitive, chat-based experience.

---

## 🚀 Features

- 🔄 **Smart Query Routing** between MySQL and Neo4j
- 📈 **Real-time Sensor Data** (temperature, occupancy) via MySQL
- 🧠 **Graph-Based Data Center Mapping** via Neo4j
- 🧾 **Natural Language to SQL/Cypher Translation** using GPT-4o
- 💬 **Friendly Conversational Interface** via Streamlit
- ✅ Handles both **technical queries** and **casual chats**

---

## 🧰 Tech Stack

- **LLM**: OpenAI GPT-4o via LangChain
- **Databases**: MySQL (relational), Neo4j (graph)
- **Frontend**: Streamlit
- **LangChain**: Prompt templates, chains, routers
- **Python Libraries**: `dotenv`, `sqlalchemy`, `re`, `mysql-connector`, `neo4j`

---

### 1. Clone the Repo

```bash
git clone https://github.com/your-username/edged-chatbot.git
cd edged-chatbot

###2. Install Dependencies
pip install -r requirements.txt
