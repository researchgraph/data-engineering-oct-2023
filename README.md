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

In order to load the data in Neo4j, it was neccesary to manipulate the original JSON file and explore it using Python. 

Activities
- Manipulate the big JSON to check the most suitable way to get chunks of data
- Get small files with chunks of data using Python script
- Create a db in Neo4j using Neo4j Desktop
- Create the queries to populate the Graph db
- Iterate through the queries via Python and Neo4j Python package

The JSON file after unzip has a size of app 4.8 GB. The file cannot be directly load into memory or by JSON even in Python. After carefully consider the options to load this big JSON, considering the use of Neo4j + APOC, and the limited resources of my local machine, I decided to go for a very non optimal way to load the data by chunks in the Neo4j DB (Desktop version 5.3.0) and using Python 🐍 for geting the chunks of JSON and to populate the DB. 

The file **test1_json2.ipynb** shows an example of a Python 🐍 code to get chunks of JSON from a big JSON file. The size of the chunks can be configurable.

For each element in the JSON file, i.e., for each paper, the following procedures in Cypher are 

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

To iterate this procedures, the Jupyter notebook **ResearchGraph4Neo4j2.ipynb** shows the Python 🐍 code where each query is built to be iterable through the small JSON files.
All the steps are done for all the generated JSON files. 

(*) Note: To make the code easy to use (and less precise), some problems caused by differences in fields between JSON elements (articles) where avoided just by skip them. For instance, some articles don't have authors information, and some authors don't have affiliation information or have a different format of affiliation (id instead of name). This issues could be addressed by a more comprehensive criteria when building the queries and using some logic to detect the differences in structures from the JSON.

(**) Note: Another option is to use the apoc.periodic.iterate method, however, it didn't work.

1.2 - Computing the values 🧮

Once the data is all in the Graph db, we can create a query to get the number of nodes per each label. The following is an example query to get the counts.

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

The resulting values are in the following table

    label:	count
    "Author":	16342
    "Organisation":	10820
    "Paper":	15612  

(*) Note: The original number of articles (papers, conferences journals, chapters, etc) is 501629. However, some of them were ignored due to the presence of problems in some fields.
This could be solved with a more tailored approach to consider ALL the posibilities in terms of fields, according to predefined criterias.


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
