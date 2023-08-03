# Neo4j Graph Query Engine

[TOC]

This document describes how to build and test a graph query engine with Neo4j database and Large Language Model (LLM).

## 1. Neo4j Environment

Firstly, you need to have a Neo4j instance. For testing, we recommend you using a docker instance by executing the following commands:

```bash
docker run \
    --name neo4j \
    -p 7474:7474 -p 7687:7687 \
    -d \
    -e NEO4J_AUTH=neo4j/neo4j \
    -e NEO4J_PLUGINS=\[\"apoc\"\]  \
    neo4j:latest
```

If you have a requirements for production, please using the [Neo4j Desktop application](https://neo4j.com/download/) instead.

## 2. Python Environment

Please install Python via [anaconda](https://www.anaconda.com/)  or [python.org](https://www.python.org/downloads/windows/), and you need install the following requirements.

```
langchain==0.0.235
neo4j==5.11.0
openai==0.27.8
```

The link to the neo4j database is

```python
graph = Neo4jGraph(
	url="bolt://localhost:7687", username="neo4j", password="neo4j"
)
```

## 3. Initialize the Neo4j database

To avoid the empty database, you can fill it with your own data by the Cypher statements, or you can execute the following command to fill it with toy data:

```
graph.query(
    """
MERGE (m:Movie {name:"Top Gun"})
WITH m
UNWIND ["Tom Cruise", "Val Kilmer", "Anthony Edwards", "Meg Ryan"] AS actor
MERGE (a:Actor {name:actor})
MERGE (a)-[:ACTED_IN]->(m)
"""
)
```

The above command is idempotent, don't worry that running it for many times.

 ## 4. Clone the Scripts

Please clone the repository from [LangchainCypher]([Suchun-sv/LangchainCypher: Linking LLM and Cypher (github.com)](https://github.com/Suchun-sv/LangchainCypher)).

```bash
git clone https://github.com/Suchun-sv/LangchainCypher.git ./LangchainCypher
```

Here, we provide our implementation of Cypher Chain, you can refer to the `./CypherChain/` to see the details.

We also provide the Basic test scripts named `connect.py` , you can provide your own endpoint:

```python
endpoint = "" # fill it with your endpoint
api_key = ""  # fill it with your api key
```

Then, using the graph cypher QA chain, you can now ask question of the graph:

```python
chain = GraphCypherQAChain.from_llm(
    ChatOpenAI(temperature=0), graph=graph, verbose=True
)
```

For example,

```python
chain.run("Who played in Top Gun?")
```

The return would be:

```bash
Generated Cypher:
MATCH (a:Actor)-[:ACTED_IN]->(m:Movie {name: 'Top Gun'}) RETURN a.name
```

You can copy the generation to the console of neo4j, or using:

```python
graph.query("MATCH (a:Actor)-[:ACTED_IN]->(m:Movie {name: 'Top Gun'}) RETURN a.name")
```

The final result would be:

```python
[{'a.name': 'Val Kilmer'},
     {'a.name': 'Anthony Edwards'},
     {'a.name': 'Meg Ryan'},
     {'a.name': 'Tom Cruise'}]
```

