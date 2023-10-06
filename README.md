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
1.1 - Get the Data into the graph db

In order to load the data in Neo4j, it was neccesary to manipulate the original JSON file, explore it using Python 🐍, and then find the way to get it into the Neo4j. The following list of activities show an overview of the performed steps:

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

The JSON file (after unzip) has a size of around 4.8 GB. This big JSON file cannot be directly load into memory or by JSON package even in Python. After carefully consider the options to load this big JSON, considering the use of Neo4j + APOC, and the limited resources of my local machine, I decided to go for a very non optimal way. I split the big JSON into smaller JSON files, and then to load the data by chunks in the Neo4j DB (Desktop version 5.3.0 + APOC), and using Python to iterate through the files and queries, in order of getting the chunks of JSON to populate the graph DB. 

The file [**Transform_BIG_JSON.ipynb**](/jnotebooks/Transform_BIG_JSON.ipynb) shows the Jupyter notebook with the Python code to get chunks of JSON from a big JSON file. The size of the chunks can be configurable.

It was considered to get 3 types of nodes:
- **AUTHOR**
- **PAPER**
- **ORGANISATION**
and 2 types of relationships:
- WRITTEN_BY: **PAPER -[WRITTEN_BY]-> AUTHOR**
- IS_PART_OF: **AUTHOR -[IS_PART_OF]-> ORGANISATION**

The next figure shows the general schema for 1 paper as node:

<img title="a title 0" alt="Alt text" src="/imag/example_papers_graph0.png">

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

The identifier for **PAPERS** were the **id** of each JSON element, for **AUTHORS** the identifier was the combination of **given name** and the **family name**, and for **ORGANISATIONS**, the identifier was the **name** of the affiliation (when available). This is not the best option, but it was the best generalisable way to call and connect all the possible nodes. The ideal scenario would be to use the DOI for each article, the ORCID for each author, and verified identifier for institutions or organisations, however, that is not the case.

To iterate these procedures, the Jupyter notebook [**ResearchGraph4Neo4j3.ipynb**](/jnotebooks/ResearchGraph4Neo4j3.ipynb) shows the Python 🐍 code where each query is built to be an iterable string, using an iterable index to call each small JSON file. All the steps were done for all the generated chunk JSON files. The image shows part of the nodes created in the graph db.

<img title="a title" alt="Alt text" src="/imag/example_papers_graph.png">

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

Using the iterative method to populate the graph db, because the low optimal approach used here, the process was able to get just **174077** records from a total of **501629** records in my local machine. The resulting values are in the following table

| Label         | Count  | 
|---------------|--------|
| Authors       | 63203  |
| Organisations | 42992  |
| Papers        | 174098 |

Knowing that these are not the total values, I double checked the values using a Python Script to analise the BIG JSON file and obtain the unique values for **PAPERS** (**id**: id or doi), **AUTHORS** (**id**: given name + family name) and **ORGANISATIONS** (**id**: name). The script is called [**explore_JSON.ipynb**](/jnotebooks/explore_JSON.ipynb) and the resulting values are:

| ITEM                 | COUNT  | 
|----------------------|--------|
| TOTAL RECORDS        | 501629 |
| UNIQUE AUTHORS      | 736857 |
| UNIQUE ORGANISATIONS | 150375 |
| UNIQUE PAPERS        | 501629 |

(*) Note: The original number of articles (papers, conferences journals, chapters, etc) is 501629. However, some of them were ignored due to the presence of problems in some fields. For instance, in some cases, inside the field **authors** some records have the affiliation as an author, which was avoided by reviewing the properties of each author in the for loops.
This could be solved with a more tailored approach to consider ALL the posibilities in terms of fields, according to predefined criterias.

For more details about construction of queries, resources, references, examples, and additional information, please, review the document [**neo4j_examples.txt**](/resources/neo4j_examples.txt).

Additional issues:
- Some articles don't have author information
- Some authors don't have affiliation information
- Some affiliations have a different format: some of them have a name, some of them have an id or url.
- Some authors don't have the complete information
- Some authors don't have a orcid id


# Update Task 1 (after the deadline)

After updating the scripts to convert BIG JSON into chunks of records and the loader (JSON->Neo4j via Python), the final numbers were 

| Label         | Count  | 
|---------------|--------|
| "Author"	| 76883 | 
| "Organisation"	| 55097 | 
| "Paper"	| 229978 | 

## Task 2
2.1 - Calculate the following measures in this data
* Top 10 organisations with the highest degree of centrality 
* Top 10 researchers with the highest degree of centrality 

Note: The main challenge in this task is understanding the structure of the network and working with centrality algorithms. 
This article can help with the algorithm: https://neo4j.com/docs/graph-data-science/current/algorithms/degree-centrality/


## Answers for Task 2 💻
1.1 - Calculate metrics for nodes

The goal is to compute the degree of centrality (doC) for nodes AUTHOR and ORGANISATION. The following list of activities show an overview of the performed steps:
- load/update the articles information into the graph db
- compute the number of connections for each node (AUTHOR/ORGANIZATION)

The code for getting the top 10 organisations' doC is

    MATCH (a:AUTHOR)-[connections:IS_PART_OF]->(o:ORGANISATION)
    WITH o, count(connections) as nconnections
    RETURN o.name as name, nconnections
    ORDER BY nconnections DESC, name DESC LIMIT 10

The values for the top 10 organisations' doC is

| Organisation | doC | 
|---------------|--------|
| "for the Comparing Alternative Ranibizumab Dosages for Safety and Efficacy in Retinopathy of Prematurity (CARE-ROP) Study Group"	| 128 | 
| "Tokyo Institute of Technology"	| 86 | 
| "Saudi Aramco"	| 83 | 
| "Graduate School of Information Science, Nara Institute of Science and Technology"	| 68 | 
| "Graduate School of Informatics, Kyoto University"	| 68 | 
| "National Institute of Informatics"	| 66 | 
| "Schlumberger"	| 59 | 
| "Graduate School of Information Science, Nagoya University"	| 59 | 
| "School of Computer, National University of Defense Technology"	| 56 | 
| "National Institute of Information and Communications Technology"	| 53 | 

The code for getting the top 10 researchers' doC is

    MATCH (p:PAPER)-[connections:WRITTEN_BY]->(a:AUTHOR)
    WITH a, count(connections) as nconnections
    RETURN a.name as name, nconnections
    ORDER BY nconnections DESC, name DESC LIMIT 10

The values for the top 10 researchers' doC is

| Researcher | doC | 
|---------------|--------|
| "Nicholas J,Wade"	| 96 | 
| "Vladik,Kreinovich"	| 82 | 
| "Abdulazeez,Abdulraheem"	| 48 | 
| "Johan,Wagemans"	| 46 | 
| "Yingxu,Wang"	| 45 | 
| "Jan J,Koenderink"	| 45 | 
| "VLADIK,KREINOVICH"	| 44 | 
| "Peter,Wenderoth"	| 35 | 
| "Salaheldin,Elkatatny"	| 34 | 
| "Daniela,Rus"	| 34 | 

(*) Note: The JSON transformation script was updated to save batches of articles. See the Jupyter Notebook with the Python code in [**Transform_BIG_JSON2.ipynb**](/jnotebooks/Transform_BIG_JSON2.ipynb). The code to populate the graph db using the batches of articles JSON files is in the Jupyter Notebook [**Batches2Neo4j.ipynb**](/jnotebooks/Batches2Neo4j.ipynb).

(*) Note: From the total of ~500k records in the original JSON file, just 229978 records (articles) were succesfully loaded into the graph db due to machine limitations. 

(*) Note: Due to some queries required lot of memory, the max memory for transations was reached. In order to solve that, the Neoj4 configuration file was modified to have a max memory of 2g for transactions (dbms.memory.transaction.total.max=2g).


## Task 3
3.1 - Visualise the graph in such a way that shows the overall scale of all the graph nodes and relationships, and highlights the major clusters.  

These are two graph visualisation tools that can be useful.
* https://gephi.org
* https://cytoscape.org

Note: The main challenge in this task is dealing with a large graph. This issue can be resolved by merging nodes or creating sub clusters. 


## Answers for Task 3 💻
3.1 - Visualise graph and highlight clusters

The objective for this part was to visualise the data and use methods for data merging or clustering tools. The following list of activities show an overview of the performed steps:
- choose a graph visualisation tool
- configuration and connect to the graph db
- load the graph and analise and visualise the network

The final obtained figure for this analysis is shown
<img title="a title" alt="Alt text" src="/imag/screenshot_234756.png">

Additional details:
- The number of records to analyse were 361958 and edges 202669
- Gephi tool was used to visualise the data. The plugin for Neo4j and plugin for metrics computation were instaled
- The following steps were performed to improve the visualisation
    - Giant component filtering (filter tools to get rid of noisy nodes)
    - Modularity calculation (statistical tools)
    - Hub calculation (statistical tools)
    - Partition of nodes using the cluster metrics
    - Size of the nodes using ranking agg metric (hub)
    - layout using force atlas 2
    - enlarge using extension
    - adding of additional details to improve the graph
 
The process results are shown in the next images. Metrics results can be found here [**Metrics**](/resources/Modularity Report.docx).

<img title="a title" alt="Alt text" src="/imag/screenshot_010035.png">

<img title="a title" alt="Alt text" src="/imag/screenshot_234619.png">

The following image show a part of the network after filtering, clustering and redistributing.

<img title="a title" alt="Alt text" src="/imag/FirstGraph.png">

(*) Note: Due machine limitations not all records were analised.

# Bonus

Complementary to the previous visualisation, Cytoscape was used to inspect the elements in the graph db just as exploratory analysis. An example image of the network after some procedures is shown. 25000 nodes were imported and filtering was applied. Clustering metrics were computed and the visualisation layout was done using circular with the computed metrics.

<img title="a title" alt="Alt text" src="/imag/cyto.png">

End of the report

