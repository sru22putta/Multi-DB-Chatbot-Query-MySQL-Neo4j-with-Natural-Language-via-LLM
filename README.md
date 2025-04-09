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


## 🗂️ Project Structure

```
📦 edged-chatbot/
├── app.py               # Main Streamlit application
├── requirements.txt     # Python dependencies
├── .env                 # API keys (not committed)
└── README.md            # Project documentation
```

---

## ⚙️ Getting Started

### 1. Clone the Repo

```bash
git clone https://github.com/your-username/chatbot.git
cd chatbot
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Set Up Environment Variables

Create a `.env` file in the root directory:

```
OPENAI_API_KEY=your_openai_api_key
```

> 💡 Optionally, move database credentials to `.env` and load them via `os.getenv()` in `app.py`.

### 4. Run the Application

```bash
streamlit run app.py
```

---

## 📊 MySQL Schema (Sensor Data)

- **Temperature Sensors**: TS101–TS106 (mapped to Rooms 101–106)  
- **Occupancy Sensors**: OS101–OS106 (mapped to Rooms 101–106)  
- **Occupancy Value**: `1` = occupied for 5 minutes, `0` = unoccupied  
- Group by `DATE(TimeStamp)` for daily aggregation  

---

## 🧠 Neo4j Graph Schema (Data Center Layout)

### Node Types

- **Room**: `room_number`, `room_type`, `temperature_profile`  
- **TemperatureSensor**: `sensor_id`, `profile`  
- **OccupancySensor**: `sensor_id`, `profile`  
- **AirConditioningUnit**: `unit_id`  

### Relationships

- `(Room)-[:HAS_SENSOR]->(Sensor)`  
- `(Sensor)-[:SENDS_DATA_TO]->(AirConditioningUnit)`  
- `(AirConditioningUnit)-[:COOLS]->(Room)`  
- `(Room {room_type: 'mechanical'})-[:HAS_UNIT]->(AirConditioningUnit)`  

---

## 🔀 How Query Routing Works

| Question Type                                 | Routed To |
|----------------------------------------------|-----------|
| Temperature, occupancy, timestamped queries  | MySQL     |
| Room, AC unit, sensor mapping, relationships | Neo4j     |
| Greetings or casual messages                 | LLM Only  |

LangChain routes the query automatically using a prompt-based router.

---

## 💬 Example Queries

- **"What’s the current occupancy of Room 102?"**  
  → Routed to MySQL → Generates SQL → Returns human-readable answer  

- **"Which AC unit cools Room 103?"**  
  → Routed to Neo4j → Generates Cypher → Returns response  

- **"Hey there!"**  
  → Detected as casual → Friendly response from LLM  

---

## 🔐 Security Tips

- ❌ Don’t hard-code database credentials in `app.py`  
- ✅ Use `.env` and environment variables  
- 🔐 Use secured, password-protected database connections  
- 🚫 Don’t expose Neo4j/MySQL endpoints publicly  

---

## 🧑‍💻 Author

**Srujan Mohan Putta**  
📍 California, USA  
🔗 [LinkedIn](https://www.linkedin.com/in/srujan-putta/)  
📧 srujanputta4@gmail.com 

---

