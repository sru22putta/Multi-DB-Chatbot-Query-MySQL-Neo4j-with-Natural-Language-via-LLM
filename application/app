import streamlit as st
import os
import re
import pandas as pd
from dotenv import load_dotenv
from deep_translator import GoogleTranslator
from langchain_core.messages import HumanMessage, AIMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
from langchain_community.utilities import SQLDatabase
from langchain_community.graphs import Neo4jGraph

# Load environment variables
load_dotenv()
os.environ['OPENAI_API_KEY'] = os.getenv("OPENAI_API_KEY")

# MySQL Setup
db_uri = "mysql+mysqlconnector://user:password@host:port/database"
db = SQLDatabase.from_uri(db_uri)
def get_mysql_schema(_): return db.get_table_info()
def run_sql_query(query): return db.run(query)

# Neo4j Setup
NEO4J_URI = "neo4j+s://<host>"
NEO4J_USERNAME = "neo4j"
NEO4J_PASSWORD = "<password>"
graph = Neo4jGraph(NEO4J_URI, NEO4J_USERNAME, NEO4J_PASSWORD)
def run_cypher_query(query): return graph.query(query)

graph_schema = graph.schema

# Utils
def clean_sql_code(code): return re.sub(r"```(?:sql)?\n*|\n*```|(?i)^sql\s*:", "", code).strip().replace("\\n", " ").replace("\n", " ")
def clean_cypher_code(code): return re.sub(r"```(?:cypher)?\n*|\n*```|(?i)^cypher\s*:", "", code).strip().replace("\\n", " ").replace("\n", " ")

def detect_language(text):
    try:
        return GoogleTranslator(source='auto', target='en').detect(text)
    except Exception:
        return "en"

def translate_to_english(text):
    return GoogleTranslator(source='auto', target='en').translate(text)

def translate_to_spanish(text):
    return GoogleTranslator(source='en', target='es').translate(text)

# LLMs
llm_casual = ChatOpenAI(model_name="gpt-3.5-turbo-0125", temperature=0).with_config({"response_format": "text"})
llm_router = ChatOpenAI(model_name="gpt-3.5-turbo-0125", temperature=0).with_config({"response_format": "text"})
llm_strong = ChatOpenAI(model_name="gpt-4o", temperature=0).with_config({"response_format": "text"})
llm_nl = ChatOpenAI(model_name="gpt-3.5-turbo-0125", temperature=0).with_config({"response_format": "text"})

# Prompts & Chains
sql_prompt = ChatPromptTemplate.from_template("""
You are an expert in SQL. Given a schema and question, write a correct SQL query that:
- Uses valid MySQL syntax
- Avoids subqueries that reference alias columns outside aggregation context
- Is compatible with sql_mode=ONLY_FULL_GROUP_BY
- Outputs only the SQL query, no explanation.
 -Occupancy Sensor - Value of 1 indicates that the room is occupied for 5 minutes, 0 indicates unoccupied. it does not indicate number of occupants
- Occupancy value 1 indicates its been occupied for a duration of 5 minutes
- There are 6 rooms max
if the value of an occupancy sensor is 1, it means it has been occupied for 5 minutes
if the question is asked about current/right now, consider the current date and time(closest to above or lower value in the interval of 5 minutes)

Notes:
- Use `forecast_occupancy` for future dates.
- Table `forecast_occupancy` has: OS101, OS102, ..., DateColumn, TimeColumn
- Each row is a 5-minute interval. A value of `1` means the room was occupied.
- If the user asks about **times a room is occupied**, return a list of continuous occupied intervals:

Schema:
{schema}

Question: {question}

SQL Query:
""")

sql_nl_prompt = ChatPromptTemplate.from_template("""
Based on the table schema below, question, SQL query, and SQL response, write a natural language response:
                                              
{schema}

Question: {question}
SQL Query: {query}
SQL Response: {response}
""")

sql_chain = (
    RunnablePassthrough.assign(schema=get_mysql_schema)
    | sql_prompt
    | llm_strong
    | StrOutputParser()
    | (lambda code: clean_sql_code(code))
)

full_sql_chain = (
    RunnablePassthrough.assign(query=sql_chain).assign(
        schema=get_mysql_schema,
        response=lambda variables: run_sql_query(variables["query"])
    )
    | sql_nl_prompt
    | llm_nl
    | StrOutputParser()
)

custom_schema_description = """
Node Types:

- Room:
    - room_number: unique room number
    - room_type: type of room such as 'dorm' or 'mechanical'
    - temperature_profile: describes which side the room is on. It can be either 'sunny' or 'shady'.

- TemperatureSensor:
    - sensor_id: unique ID of the temperature sensor
    - unit: Celsius
    - profile: indicates whether sensor is in a sunny or shady area

- OccupancySensor:
    - sensor_id: unique ID of the occupancy sensor
    - profile: full_time_student or night_student

- AirConditioningUnit:
    - unit_id: unique ID of the AC unit

Nodes 
Rooms: 101, 102, 103, 104, 105, 106, 201, 202
Occupancy Sensors: OS101, OS102, OS103, OS104, OS105, OS106
Temperature Sensors: TS101, TS102, TS103, TS104, TS105, TS106
Air Conditioning Units: ACU1, ACU2

Air conditioning Unit 1 is connected to rooms 101, 102, 103
Air conditioning unit 2 cools rooms 104, 105, 106
each room has 1 occupancy sensor and 1 temperature sensor
Relationships:

- **(Room {room_type: 'dorm'})-[:HAS_SENSOR]->(TemperatureSensor)**: Indicates that a dorm room is equipped with a temperature sensor.
  
- **(Room {room_type: 'dorm'})-[:HAS_SENSOR]->(OccupancySensor)**: Indicates that a dorm room is equipped with an occupancy sensor.

- **(AirConditioningUnit)-[:COOLS]->(Room {room_type: 'dorm'})**: Connects each air conditioning unit with the dorm room(s) it cools, indicating a direct functional relationship.

- **(TemperatureSensor)-[:SENDS_DATA_TO]->(AirConditioningUnit)**: Establishes that temperature sensors provide/sends data/reports directly to air conditioning units to regulate temperature based on real-time data.

- **(OccupancySensor)-[:SENDS_DATA_TO]->(AirConditioningUnit)**: Shows that occupancy sensors provide/sends data/reports occupancy status to air conditioning units, aiding in energy-efficient temperature regulation.

- **(Room {room_type: 'mechanical'})-[:HAS_UNIT]->(AirConditioningUnit)-[:COOLS]->(Room {room_type: 'dorm'})**: This complex relationship chain highlights that mechanical rooms house air conditioning units which in turn cool dorm rooms. It underscores the indirect relationship between mechanical rooms and dorm rooms via air conditioning systems.

This schema is designed to clearly delineate both the physical and functional connections within the system, ensuring that queries regarding the relationships between rooms and units can be accurately and consistently handled.
"""

cypher_prompt = ChatPromptTemplate.from_messages([
    ("system", 
     """
You are a Cypher expert. Given a user question and a graph schema, return ONLY the raw Cypher query (no explanation or markdown).

Instructions:
- Output only the Cypher query
- Use the correct relationship directions
- Always compare sensor_id or unit_id using `toLower()` for case-insensitive matching
- If the question is about which AC unit a sensor sends data to:
  → MATCH (s:Sensor)-[:SENDS_DATA_TO]->(ac:AirConditioningUnit)

- If the question is about which AC unit a room's sensors send data to:
  → MATCH (r:Room)-[:HAS_SENSOR]->(s)-[:SENDS_DATA_TO]->(ac:AirConditioningUnit)
  
- AC unit/Air conditioning unit identifiers always start with 'ACU'. If a number like "1" is mentioned, treat it as "ACU1".
- Always compare unit IDs using `toLower(ac.unit_id)` and format user inputs similarly, e.g., `toLower('acu1')`.

- Make sure to include DISTINCT if multiple sensors in the same room send to the same AC
- Always return `ac.unit_id`


Schema:
{custom_schema_description}
{graph_schema}    
"""
    ),
    ("human", "Question: {question}\nCypher Query:")
])

cypher_nl_prompt = ChatPromptTemplate.from_template("""
Based on the schema, user's question, Cypher query, and the query response, give a natural language answer.

Schema:
{custom_schema_description}
{graph_schema}                                                    

Question: {question}
Cypher Query: {query}
Cypher Response: {response}

Answer:
""")

cypher_chain = (
    RunnablePassthrough.assign(custom_schema_description=lambda _: custom_schema_description,
        graph_schema=lambda _: graph_schema,)
    | cypher_prompt
    | llm_strong
    | StrOutputParser()
    | (lambda code: clean_cypher_code(code))
)

full_neo4j_chain = (
    RunnablePassthrough.assign(query=cypher_chain).assign(
        custom_schema_description=lambda _: custom_schema_description,
        graph_schema=lambda _: graph_schema,
        query=lambda vars: clean_cypher_code(vars["query"]),
        response=lambda vars: run_cypher_query(vars["query"])
    )
    | cypher_nl_prompt
    | llm_nl
    | StrOutputParser()
)

router_prompt = ChatPromptTemplate.from_template("""
You are a smart query router. Decide whether the user's question should be handled by a **SQL database** or a **graph database (Neo4j)**.

Use the following rules:
- If about sensor readings, timestamps, temperature, occupancy, trends, forecasts, or predictions → "mysql"
- If about room relationships, AC unit mappings → "neo4j"

Respond with one word: mysql or neo4j

Question: {question}
""")

router_chain = router_prompt | llm_router  | StrOutputParser()

casual_prompt = ChatPromptTemplate.from_template("You are a friendly assistant. Respond casually to: {query}")
casual_chain = casual_prompt | llm_casual  | StrOutputParser()

def is_casual_question(query):
    check_prompt = ChatPromptTemplate.from_template(
        "Is the following user query a general greeting or casual question not related to a database? Answer yes or no.\n\nQuery: {query}"
    )
    chain = check_prompt | llm_router  | StrOutputParser()
    response = chain.invoke({"query": query})
    return "yes" in response.lower()

def unified_llm_chain(question: str):
    db_source = router_chain.invoke({"question": question}).strip().lower()
    if db_source == "mysql":
        return full_sql_chain.stream({"question": question})
    elif db_source == "neo4j":
        return full_neo4j_chain.stream({"question": question})
    return iter(["❌ Could not determine the database."])

# Streamlit UI
st.set_page_config(page_title="Chatbot")
st.title("📊 Chatbot")


if "chat_history" not in st.session_state:
    st.session_state.chat_history = []

for message in st.session_state.chat_history:
    role = "Human" if isinstance(message, HumanMessage) else "AI"
    with st.chat_message(role):
        st.markdown(message.content)

user_query = st.chat_input("Ask something...")
if user_query:
    st.session_state.chat_history.append(HumanMessage(user_query))
    with st.chat_message("Human"):
        st.markdown(user_query)

    with st.chat_message("AI"):
        original_lang = detect_language(user_query)
        is_spanish = original_lang.lower() == "spanish"

        query_en = translate_to_english(user_query) if is_spanish else user_query

        if is_casual_question(query_en):
            ai_stream = casual_chain.stream({"query": query_en})
        else:
            ai_stream = unified_llm_chain(query_en)

        from openai import BadRequestError 

        try:
            ai_response = "".join(ai_stream)
            final_response = translate_to_spanish(ai_response) if is_spanish else ai_response
            st.markdown(final_response)
        except BadRequestError as e:
            if "context length" in str(e).lower():
                final_response = "⚠️ Sorry, I couldn't process that request because it exceeds the model's memory limit."
                st.markdown(final_response)
            else:
                raise e


    st.session_state.chat_history.append(AIMessage(final_response))
