# Data Engineering Tasks
Oct 2023

Deadlines

Task 1 - Wednesday 4th Oct. 5pm (AEST)

Task 2 & 3 - Friday 6th Oct. 5pm (AEST)



## Instruction
* Make a fork from this repository.
* Complete the following tasks, commit and push the outcomes to the fork.
* Update this README file to provide information about the added files and instructions on using them. 


## Task 1
1.1 - Export the JSON file to a database including MongoDB, or SQL, or Neo4j

1.2 - Write a Jupyter Notebook that uses the file in the database to calculate the following
* Number of Articles
* Number of Organisations (Deduplicated Affiliations)
* Number of Researchers

Note: you only need to commit the notebook, and you do not need to provide a backup of the database


## Answers for Task 1 💻
1.1 - Data into the db

In order to load the data in Neo4j, it was neccesary to manipulate the original JSON file, explore it using Python, and then find the way to push it into the Neo4j. The following list of activities show an overview of the performed steps:

Activities
- Manipulate the big JSON to check the most suitable way to get chunks of data
- Get small files with chunks of data using Python script
- Create a db in Neo4j using Neo4j Desktop
- Create the queries to populate the Graph db
- Generate a Python code to connect to the graph db and perform the queries
- Iterate through the queries via Python and Neo4j Python package
- Populate and create the nodes for **AUTHORS**, **PAPERS**, and **ORGANISATIONS**
- Create the relationships:
    - **PAPER -[WRITTEN_BY]-> AUTHOR**
    - **AUTHOR -[IS_PART_OF]-> ORGANISATION**

The JSON file (after unzip) has a size of around 4.8 GB. This big JSON file cannot be directly load into memory or by JSON package even in Python. After carefully consider the options to load this big JSON, considering the use of Neo4j + APOC, and the limited resources of my local machine, I decided to go for a very non optimal way. I split the big JSON into smaller JSON files, and then to load the data by chunks in the Neo4j DB (Desktop version 5.3.0 + APOC), and using Python 🐍 to iterate through the files and queries, in order of getting the chunks of JSON to populate the graph DB. 

The file [**Transform_BIG_JSON.ipynb**](/Transform_BIG_JSON.ipynb) shows the Jupyter notebook with the Python 🐍 code to get chunks of JSON from a big JSON file. The size of the chunks can be configurable.

It was considered to get 3 types of nodes:
- AUTHOR
- PAPER
- ORGANISATION
and 2 types of relationships:
- WRITTEN_BY: PAPER -[WRITTEN_BY]-> AUTHOR
- IS_PART_OF: AUTHOR -[IS_PART_OF]-> ORGANISATION

In the case of 1 item (article) per JSON the reading and creation of nodes is done. For each JSON file (eg. paper_1.json),  i.e., for each article, the following procedures were performed using Cypher:

To create the nodes for PAPERS:

    CALL apoc.load.json("file:///paps/paper_1.json")
    YIELD value
    WITH value.author AS author
    RETURN author
    
To create the nodes for ORGANISATIONS:

    CALL apoc.load.json("file:///paps/paper_1.json")
    YIELD value
    WITH value.author AS authors
    UNWIND authors AS au
    UNWIND au.affiliation as affiliation
    MERGE (o:ORGANISATION {name: affiliation.name})
    RETURN o

To create the nodes for AUTHORS and the relationships with PAPERS and ORGANISATIONS:

    CALL apoc.load.json("file:///paps/paper_1.json")
    YIELD value
    WITH value.author AS authors, value.id as code
    UNWIND authors AS au
    UNWIND au.affiliation as affiliation
    MERGE (a:AUTHOR {name: COALESCE(au.given ,"") + ',' + COALESCE(au.family ,"")}) ON CREATE SET a.given = au.given, a.family = au.family, a.affiliation = affiliation.name           
    MERGE (p:PAPER {name: code})
    MERGE (o:ORGANISATION {name: affiliation.name})
    MERGE (p)-[:WRITTEN_BY]->(a)
    MERGE (a)-[:IS_PART_OF]->(o)
    RETURN a, p, o

The identifier for PAPERS were the id of each JSON element, for AUTHORS the identifier was the combination of given name and family name, and for ORGANISATIONS, the identifier was the name of the affiliation. This is not the best option, but it was the best generalisable way to call and connect the nodes. To iterate these procedures, the Jupyter notebook  [**ResearchGraph4Neo4j3.ipynb**](/ResearchGraph4Neo4j3.ipynb) shows the Jupyter notebook with the Python 🐍 code where each query is built to be an iterable string, using an iterable index to call each small JSON file. All the steps are done for all the generated JSON files. The image shows part of the nodes created in the graph db.

<img title="a title" alt="Alt text" src="/example_papers_graph.png">

(*) Note: To make the code easy to use (and less precise), some issues caused by differences in fields between JSON elements (articles) were avoided just by skipping them. For instance, some articles don't have authors' information, and some authors don't have affiliation's information or they have a different format for the affiliation (eg. 'id' instead of 'name'). This issues could be addressed by a more comprehensive criteria when building the queries and using some better logic to detect the differences in structures from the JSON.

(**) Note: Another option to load a BIG JSON in an iteratively way is to use the apoc.periodic.iterate method, however, it didn't work in my local.

1.2 - Computing the values 🧮

Once the data is all in the Graph db, we can create a query to get the number of nodes per each label. The following is an example query to get the counts for each type of node:

    MATCH (a:AUTHOR)
    WITH count(a) AS count
    RETURN 'Author' AS label, count
    UNION ALL
    MATCH (o:ORGANISATION)
    WITH count(o) AS count
    RETURN 'Organisation' AS label, count
    UNION ALL
    MATCH (p:PAPER)
    WITH count(p) AS count
    RETURN 'Paper' AS label, count

Using the iterative method to populate the graph db, because the low optimal approach used here, the process was able to get just 168116 records from a total of 501629 records in my local machine. The resulting values are in the following table

| label         | count  | 
|---------------|--------|
| Authors       | 60346  |
| Organisations | 41253  |
| Papers        | 167650 |

Knowing that these are not the total values, I double check the values using a Python Script to analise the BIG JSON file and obtain the unique values for PAPERS (id: id or doi), AUTHORS (id: given name + family name) and ORGANISATIONS (id: name). The script is called [**explore_JSON.ipynb**](/explore_JSON.ipynb) and the resulting values are:

| ITEM                 | COUNT  | 
|----------------------|--------|
| TOTAL RECORDS        | 501629 |
|  UNIQUE AUTHORS      | 736857 |
| UNIQUE ORGANISATIONS | 150375 |
| UNIQUE PAPERS        | 501629 |

(*) Note: The original number of articles (papers, conferences journals, chapters, etc) is 501629. However, some of them were ignored due to the presence of problems in some fields. For instance, in some cases, inside the field **authors** some records have the affiliation as an author, which was avoided by reviewing the properties of each author in the for loops.
This could be solved with a more tailored approach to consider ALL the posibilities in terms of fields, according to predefined criterias.

For more details about construction of queries, resources, references, examples, and additional information, please, review the document [**neo4j_examples.txt**](/neo4j_examples.txt).


## Task 2
2.1 - Calculate the following measures in this data
* Top 10 organisations with the highest degree of centrality 
* Top 10 researchers with the highest degree of centrality 

Note: The main challenge in this task is understanding the structure of the network and working with centrality algorithms. 
This article can help with the algorithm: https://neo4j.com/docs/graph-data-science/current/algorithms/degree-centrality/



## Task 3
3.1 - Visualise the graph in such a way that shows the overall scale of all the graph nodes and relationships, and highlights the major clusters.  

These are two graph visualisation tools that can be useful.
* https://gephi.org
* https://cytoscape.org

Note: The main challenge in this task is dealing with a large graph. This issue can be resolved by merging nodes or creating sub clusters. 
