# 🤖 Multi-DB Chatbot: Query MySQL & Neo4j with Natural Language via LLM

An LLM-powered chatbot that lets users query real-time sensor data (temperature, occupancy) from MySQL and explore data center layouts via Neo4j. It also supports forecasting queries and Spanish-English translation using Google Translator. Built with Streamlit, LangChain, and OpenAI's GPT-4o, this chatbot intelligently routes queries to the appropriate backend and returns natural language responses.

---

## 🚀 Features

- 🔄 Smart Query Routing between MySQL and Neo4j  
- 🌡️ Real-time Sensor Data (temperature, occupancy) via MySQL  
- 🧠 Graph-Based Data Center Mapping via Neo4j  
- 📈 Forecasting Support (Future occupancy predictions)  
- 🌍 Spanish-English Input and Output Translation  
- 💬 Friendly Conversational Interface via Streamlit  
- 🤖 Natural Language to SQL/Cypher using GPT-4o  
- ✅ Handles both technical queries and casual chats  

---

## 🧰 Tech Stack

- **LLM**: OpenAI GPT-4o via LangChain  
- **Databases**: MySQL (relational), Neo4j (graph)  
- **Frontend**: Streamlit  
- **LangChain**: Prompt templates, chains, routers  
- **Translation**: Google Translator (via `deep-translator`)  
- **Python Libraries**: `dotenv`, `sqlalchemy`, `re`, `pandas`, `deep-translator`, `mysql-connector`, `neo4j`

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

## 🌐 Language Support

The chatbot auto-detects if the user input is in Spanish and:
- Translates it to English before querying
- Translates the English response back to Spanish

---

## 📊 MySQL Schema (Sensor + Forecasting Data)

- **Temperature Sensors**: TS101–TS106 → Rooms 101–106  
- **Occupancy Sensors**: OS101–OS106 → Rooms 101–106  
- **Forecast Table**: `forecast_occupancy` (OS101–106, DateColumn, TimeColumn)  
- **Occupancy Value**: 1 = occupied for 5 mins, 0 = unoccupied  
- Group by `DATE(TimeStamp)` for daily trends  

---

## 🧠 Neo4j Graph Schema (Data Center Layout)

### 🌐 Graph Database Visualization (Neo4j Schema)

![Graph Database](./Graph%20Database.png)

### Node Types

- `Room`: room_number, room_type, temperature_profile  
- `TemperatureSensor`: sensor_id, profile  
- `OccupancySensor`: sensor_id, profile  
- `AirConditioningUnit`: unit_id  

### Relationships

- `(Room)-[:HAS_SENSOR]->(Sensor)`  
- `(Sensor)-[:SENDS_DATA_TO]->(AirConditioningUnit)`  
- `(AirConditioningUnit)-[:COOLS]->(Room)`  
- `(Room {room_type: 'mechanical'})-[:HAS_UNIT]->(AirConditioningUnit)`  

---

## 🔀 How Query Routing Works

| Question Type                                  | Routed To |
|-----------------------------------------------|-----------|
| Sensor data, occupancy, temperature, forecast | MySQL     |
| Room relationships, AC mappings               | Neo4j     |
| Greetings/casual questions                    | LLM Only  |

### 🧠 LangChain Architecture

![LangChain Architecture](./image.png)

---

## 💬 Example Queries

- **"¿Está ocupada la habitación 102 ahora?"** (Spanish)  
  → Translates → Routed to MySQL → SQL generated → Natural language response in Spanish  

- **"Which rooms are cooled by ACU2?"**  
  → Routed to Neo4j → Cypher generated → Human-readable response  

- **"Predict occupancy in Room 105 tomorrow morning."**  
  → Routed to MySQL → Forecast table → SQL query → Returns prediction  

- **"Hi there!"**  
  → Friendly casual reply from the LLM

  ### 🔁 Result Snapshot

![Result](./result%20snapshot%20(translation).png)

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
