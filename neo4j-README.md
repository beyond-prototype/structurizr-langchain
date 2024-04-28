## Neo4j

### 1. Neo4j APOC plugin

Please make sure the compatable version of APOC plugin is installed in Neo4j. Otherwise, the error will be raised due to either APOC plugin version is not compatible with neo4j version or the `apoc.meta.data()` procedure is not allowed in neo4j configuration.

```log
ValueError: Could not use APOC procedures. Please ensure the APOC plugin is installed in Neo4j and that 'apoc.meta.data()' is allowed in Neo4j configuration 
```

According to [APOC documentation](https://neo4j.com/labs/apoc/4.4/overview/apoc.meta/apoc.meta.data/), `apoc.meta.data()` is only avaialble under APOC verison 4.4.

Please check [compatable version of Neo4j and APOC](https://neo4j-contrib.github.io/neo4j-apoc-procedures/versions.json) are installed.

This is the compatible version of APOC plugin for Neo4j 4.4.x used in this project.
```json
{
        "neo4j": "4.4.32",
        "neo4jVersion": "4.4.32",
        "apoc": "4.4.0.26",
        "version": "4.4.0.26",
        "url": "http://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.4.0.26",
        "homepageUrl": "http://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/4.4.0.26",
        "jar": "https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/4.4.0.26/apoc-4.4.0.26-all.jar",
        "downloadUrl": "https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/4.4.0.26/apoc-4.4.0.26-all.jar",
        "sha1": "a7a393978458aa301988fc0b8ea8f33213b03cc8",
        "sha256": "f24d993d981a330757ca59e3b117b57f8fca70329e4ffaf5a435ed80309f49b0",
        "md5": "5a42a32e12432632124acd682382c91d",
        "config": {
            "+:dbms.security.procedures.unrestricted": [
                "apoc.*"
            ]
        }
    }
```

### 2. Neo4j docker compose

Create [neo4j-compose.yml](neo4j-compose.yml) and configure the volume for neo4j data, logs, configuration and plugins.

```yml
services:
  neo4j:
    image: neo4j:4.4.32
    container_name: neo4j
    volumes:
     - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/neo4j/data:/data:rw
     - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/neo4j/logs:/logs:rw
     - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/neo4j/conf:/var/lib/neo4j/conf:rw
     - ${DOCKER_VOLUME_DIRECTORY:-.}/volumes/neo4j/plugins:/plugins
    ports:
      - "7474:7474"
      - "7687:7687"
    environment:
      - NEO4J_AUTH=neo4j/neo4j123
```

### 3. Configure APOC plugin

Download `https://github.com/neo4j-contrib/neo4j-apoc-procedures/releases/download/4.4.0.26/apoc-4.4.0.26-all.jar` to the local volume for neo4j plugins `/plugins/apoc-4.4.0.26-all.jar`

Create  `neo4j.conf` under the volume for neo4j configuration and enable all apoc procedures.
```
dbms.security.procedures.unrestricted=apoc.*
```

### 4. Start Neo4j server

```bash
docker compose -f neo4j-compose.yml up -d
```

Open neo4j browser [http://localhost:7474/browser/](http://localhost:7474/browser/) and check the APOC plugin version.

```bash
neo4j$ return apoc.version()
```

The following text output will be returned.
```
╒══════════════╕
│apoc.version()│
╞══════════════╡
│"4.4.0.26"    │
└──────────────┘
```

### 5. Structurizr model to Neo4j

The project below is to convert the Structurizr workspace to Neo4j graph.

https://github.com/beyond-prototype/structurizr-neo4j

### 6. Langchain Neo4jGraph

```python
from langchain_community.graphs import Neo4jGraph
from langchain.chains import GraphCypherQAChain

graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="neo4j123")

graph.query("MATCH (n:Person) RETURN n") 
```

```json
[{'n': {'name': 'Personal Banking Customer',
   'description': 'A customer of the bank, with personal bank accounts.'}},
 {'n': {'name': 'Customer Service Staff',
   'description': 'Customer service staff within the bank.'}},
 {'n': {'name': 'Back Office Staff',
   'description': 'Administration and support staff within the bank.'}}]
```


### 7. References

https://python.langchain.com/docs/use_cases/graph/

https://python.langchain.com/docs/integrations/graphs/neo4j_cypher/

https://python.langchain.com/docs/use_cases/graph/prompting/#setup

https://python.langchain.com/docs/integrations/vectorstores/neo4jvector/
