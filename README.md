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
