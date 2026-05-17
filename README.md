<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:0d0d1a,40:1a1a3e,70:2d2d6b,100:0d0d1a&height=220&section=header&text=LogicGraph&fontSize=72&fontColor=ffffff&fontAlignY=38&fontStyle=bold&desc=From%20Prolog%20Facts%20to%20Graph%20Relationships&descSize=18&descAlignY=62&descColor=a78bfa" width="100%"/>

<br/>

<p align="center">
  <img src="https://img.shields.io/badge/Prolog-Symbolic%20AI-8B6914?style=for-the-badge&logo=prolog&logoColor=white"/>
  <img src="https://img.shields.io/badge/Neo4j-Graph%20Database-008CC1?style=for-the-badge&logo=neo4j&logoColor=white"/>
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/pytholog-Prolog%20Bridge-a78bfa?style=for-the-badge"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Domain-Knowledge%20Engineering-2d2d6b?style=flat-square"/>
  <img src="https://img.shields.io/badge/Input-Prolog%20.pl%20File-orange?style=flat-square"/>
  <img src="https://img.shields.io/badge/Output-Neo4j%20Graph-008CC1?style=flat-square"/>
  <img src="https://img.shields.io/badge/Query-Bolt%20Protocol-green?style=flat-square"/>
</p>

<br/>

> **LogicGraph** is a Python tool that migrates a Prolog symbolic knowledge base into a Neo4j graph database — parsing `.pl` facts and rules, extracting entities and relationships via regex, creating typed nodes via Cypher, and wiring them with directional graph edges. Two AI paradigms, one bridge.

<br/>

**[🧠 The Concept](#-the-concept) · [⚙️ How It Works](#️-how-it-works) · [🔄 Conversion Pipeline](#-conversion-pipeline) · [🚀 Quickstart](#-quickstart) · [📁 File Reference](#-file-reference)**

</div>

---

## 🧠 The Concept

Prolog and Neo4j represent two fundamentally different approaches to storing and reasoning over structured knowledge.

| | Prolog | Neo4j |
|---|---|---|
| **Paradigm** | Symbolic AI, Logic Programming | Property Graph Database |
| **Knowledge unit** | Facts and rules (`.pl` clauses) | Nodes and relationships |
| **Query language** | Prolog Horn clauses | Cypher |
| **Reasoning** | Backward chaining inference | Graph traversal |
| **Best for** | Rule-based deduction | Relationship-first querying |

Millions of legacy expert systems were built in Prolog throughout the 1980s and 1990s. Modern knowledge graphs — the kind powering Google's Knowledge Graph, pharmaceutical drug databases, and enterprise ontologies — live in graph databases like Neo4j. LogicGraph bridges that gap, enabling Prolog knowledge bases to be migrated, visualised, and queried with modern graph tooling.

```
parent(tom, bob).          →     (tom)-[:is_a_parent_of]->(bob)
male(tom).                 →     (:Person {name:'tom', gender:'male'})
female(mary).              →     (:Person {name:'mary', gender:'female'})
```

---

## ⚙️ How It Works

### Step 1 — Read the Prolog Knowledge Base

The `.pl` file is loaded using `pytholog`, a Python interface to Prolog's inference engine. This initialises a `KnowledgeBase` object that can later be queried using Prolog-style unification.

```python
knowledge_base = KnowledgeBase('Family Network')
knowledge_base.from_file('family-relation.pl')
```

### Step 2 — Extract Facts and Rules via Regex

The `.txt` companion file is parsed line by line. A regex pattern distinguishes Prolog rules (which contain `:-`) from plain facts:

```python
pattern = r'^\w+\(.*?\)\s*:-.*?\.'   # matches rules like: grandparent(X,Z) :- parent(X,Y), parent(Y,Z).
```

Lines matching the pattern go to `rules`. Everything else (ground facts like `male(tom).`) goes to `facts`.

### Step 3 — Create Neo4j Nodes

Each fact is parsed to extract its predicate (the relationship/type) and argument (the entity name). These map directly to Neo4j node properties:

```python
# Prolog fact:   male(tom).
# Cypher result: CREATE (n:Person {name: 'tom', gender: 'male'})

predicate → gender property
argument  → name property
```

### Step 4 — Create Neo4j Relationships

Parent-child relationships are extracted from facts matching the `parent(X, Y)` pattern and converted into directed Cypher edges:

```python
# Prolog fact:   parent(tom, bob).
# Cypher result:
MATCH (p:Person {name: 'tom'}), (c:Person {name: 'bob'})
CREATE (p)-[:is_a_parent_of]->(c)
```

---

## 🔄 Conversion Pipeline

```
family-relation.pl  (Prolog source)
         │
         │  pytholog.KnowledgeBase.from_file()
         ▼
  Prolog Knowledge Base loaded in memory
  Available for Prolog-style inference queries
         │
family-relation.txt  (plain text export)
         │
         │  Regex pattern matching per line
         ▼
  ┌──────────────┬──────────────────┐
  │    Facts     │      Rules       │
  │  male(tom).  │  grandparent(X) │
  │ parent(t,b). │  :- parent(X,Y) │
  └──────┬───────┴──────────────────┘
         │
         │  get_predicate_and_name()
         ▼
  Parse each fact → predicate + argument
         │
         ├──────────────────────────────────────┐
         ▼                                      ▼
  create_nodes()                    create_relationships()
  Cypher CREATE per person          Regex match parent(X,Y)
  {:name, :gender}                  Cypher MATCH + CREATE edge
         │                                      │
         └──────────────┬───────────────────────┘
                        ▼
              Neo4j Graph Database
              Nodes + Directed Edges
              Queryable via Cypher
```

---

## 🔬 Prolog Input Example

```prolog
% family-relation.pl

male(tom).
male(bob).
male(pat).
female(liz).
female(ann).
female(pat).

parent(tom, bob).
parent(tom, liz).
parent(bob, ann).
parent(bob, pat).

grandparent(X, Z) :- parent(X, Y), parent(Y, Z).
```

## 🌐 Neo4j Output Example

After conversion the graph looks like this in Neo4j Browser:

```
(tom:Person {gender:'male'}) -[:is_a_parent_of]-> (bob:Person {gender:'male'})
(tom:Person {gender:'male'}) -[:is_a_parent_of]-> (liz:Person {gender:'female'})
(bob:Person {gender:'male'}) -[:is_a_parent_of]-> (ann:Person {gender:'female'})
(bob:Person {gender:'male'}) -[:is_a_parent_of]-> (pat:Person {gender:'female'})
```

You can then run Cypher queries that mirror Prolog inference:

```cypher
-- Find all grandchildren of tom
MATCH (tom:Person {name:'tom'})-[:is_a_parent_of*2]->(grandchild)
RETURN grandchild.name
```

---

## 📁 File Reference

| File | Language | Purpose |
|---|---|---|
| `main.py` | Python | Core converter — `FamilyNetwork` class with all five pipeline methods |
| `family-relation.pl` | Prolog | Source knowledge base with facts and inference rules |
| `family-relation.txt` | Text | Plain text export of the Prolog file for regex-based parsing |

---

## 🚀 Quickstart

### 1. Clone the repository

```bash
git clone https://github.com/mahmedmajeedai/Converting-Prolog-Knowledge-base-to-Neo4j-Graph-Database.git
cd Converting-Prolog-Knowledge-base-to-Neo4j-Graph-Database
```

### 2. Install dependencies

```bash
pip install neo4j pytholog
```

### 3. Start Neo4j

Download and start [Neo4j Desktop](https://neo4j.com/download/) or run it via Docker:

```bash
docker run \
  --publish=7474:7474 --publish=7687:7687 \
  --env=NEO4J_AUTH=neo4j/12345678 \
  neo4j:latest
```

### 4. Configure and run

Update the connection details in `main.py` if needed:

```python
uri  = "bolt://localhost:7687"
auth = ('neo4j', 'your_password')
```

Then run:

```bash
python main.py
```

### 5. Explore in Neo4j Browser

Open `http://localhost:7474` in your browser and run:

```cypher
MATCH (n) RETURN n
```

The entire family graph is now visualised as an interactive node-edge diagram.

---

## 🔧 Extending to Other Knowledge Domains

LogicGraph was implemented using a family relations domain, but the architecture is domain-agnostic. The same pipeline can convert any Prolog knowledge base where:

- Ground facts define entities and their types
- Relational predicates define edges between entities

Example domains:

| Domain | Prolog Facts | Neo4j Graph |
|---|---|---|
| **Medical ontology** | `treats(aspirin, headache).` | Drug → treats → Condition |
| **Company hierarchy** | `reportsTo(alice, bob).` | Employee → reportsTo → Manager |
| **Geographic data** | `locatedIn(paris, france).` | City → locatedIn → Country |
| **Academic network** | `supervisedBy(student, prof).` | Student → supervisedBy → Professor |

---

## 📄 License

This project is open source. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Built by [Muhammad Ahmed Majeed](https://github.com/mahmedmajeedai)**

*Bridging symbolic AI and modern knowledge graphs*

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:2d2d6b,50:1a1a3e,100:0d0d1a&height=100&section=footer" width="100%"/>

</div>
