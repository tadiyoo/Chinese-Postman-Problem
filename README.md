# Satellite knowledge graph and Q&A based on neo4j, django, pytorch, py2neo
     The functions mainly include entity recognition, entity search, relationship search and question & answer modules.
     The satellite datasets used in the project comes from online public UCS-Database datasets
    

## Project structure
```
.
├── Prepared_data
│   ── custom_dict // General data
│   ── node  // Satellite node data
│   ── relation   // Satellite relation data
│   ── CYPHER4csvImport.txt   // Cypher query to create the nodes and relationships
│
├── img //Contains all the output images
│
├── QA_Classification
│   ── classification_data //Intent recognition training data
│   ── pytorch  // Neural network classification model
│       
├── Satellite_KnowledgeGraph     // django project path
│   ├── Neo4J_Module  // Model layer, used to interact with neo4j and realize core functions such as query
│   ├── Satellite_KnowledgeGraph   // The logic used to write the page(View)
│   ├── static    // Static resources
│   ├── templates   // html page
│   └── Models_util   // Including pre-loading some data and models
.
```

## Prepare data and build entities and relationships
Note: The following data import is done on the Neo4j console. Put all CSV files of folder data/node and data/relation into the Import folder under the neo4j installation directory:

1. Neo4j Configuration

- Database Name：Satellite_KG
- Username：neo4j
- Password：12345

2. CYPHER4csvImport.txt: Input the all statements on by one in this file on the Neo4j console and run

3. After importing, you will see:

    - 10 Node Labels: (satellites, CntryUNRegistry, Operator, Users, Purpose, Operator, OrbitsClassAndType, Contractor, LaunchSite, LaunchVehicle, SatQA)
    - 27 Relationship Types: (SatsUNRegistry, SatsOperator, SatsUser, SatsPurpose, SatsOrbit, SatsContractor, SatsLaunchSite, SatsLaunchVehicle, UNRegsOperator, UNRegsUser, UNRegsPurpose, UNRegsContractor, UNRegsLaunchSite, OperatorsUser, OperatorsPurpose, OperatorsContractor, OperatorsLaunchSite, UsersPurpose, UsersOrbit, UsersContractor, UsersLaunchSite, UsersLaunchVehicle, PurposesOrbit, PurposesContractor, PurposesLaunchSite, PurposesLaunchVehicle, ContractorsLaunchSite)


##### Node creation sample:
    4,840 entities created from 9 node labels and here the following figure shows all those labels and the sample cypher query of entity creation:
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/node_labels.png)


##### Relationship creation sample:
    35,999 relationships created from 27 relationship types and here the following figure shows all those types and the sample cypher query of relationship creation:
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/relationship.png)


### 5. Test the constructed satellite knowledge graph
    I use Cypher Query Language to extract data from neo4j database. To ensure that the knowledge graph created works as intended, I have tested by doing queries to observe the relationship between entities and give an answer to the following questions: 

1. List all the satellites that China operates or owns, and what are they used for?
    Cypher query used to get the answer for this question:

    MATCH (s: Satellites) - [rel1: SatsOperator] -> (o: Operator {Country: "China"}) - [rel2: OperatorsPurpose] -> (p: Purpose) RETURN s, o, p

The query output is below:
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/KGQ1.png)

2. Find all the possible relationships of the earth observation satellites with other remained node labels where the primarily users are Military.
    Cypher query used to get the answer for this question:

    MATCH (u: Users {name: “Military"}) - [rel1: UsersPurpose] - (p: Purpose {name: “Earth Observation"}) - [rel2: OperatorsPurpose] - (o: Operator) - [rel3: UNRegsOperator] -(ct: CntryUNRegistry) - [rel4: UNRegsContractor] - (c: Contractor) - [rel5: ContractorsLaunchSite] - > (ls: LaunchSite) - [rel6: SatsLaunchSite] - (s: Satellites) - [rel7: SatsOrbit] -> (oct: OrbitsClassAndType) RETURN u, p, o, ct, c, ls, s, oct
    LIMIT 1000

The query output is below:
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/KGQ2.png)


## functional module

Local start command：python manage.py runserver (or Satellite_KnowledgeGraph\run.bat)

turn on：http://127.0.0.1:8000/

### 1. Entity Recognition
    Mainly identify people, place names, institutions, satellite names and owner or operator countries name
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/ner.png)

### 2. Entity Search
    Entering the name of of the satellite/user/pupose/contractor/launch site/launch vehicle/operator/owner/country of UN registry/class/type of orbit, it will show the direct relationship
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/search_ner.png)

### 3. Relationship Search
    Main entity relationships: SatsUNRegistry, SatsOperator, SatsUser, SatsPurpose, SatsOrbit, SatsContractor, SatsLaunchSite, SatsLaunchVehicle, and others.

    You can choose different relationship types and enter the name of the satellite/user/pupose/contractor/launch site/launch vehicle/operator/owner/country of UN registry/class/type of orbit, and show it to a detail graph
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/search_relation.png)

### 4. Satellite Q&A

    1.Use classification models to recognize user input
    
    (1).The training data is in the directory intent_classification\classification_data\classification_segment_data.txt
    
    (2).A total of 935 intent categories, see catalog intent_classification\classification_data\question_classification.txt

    (3).English word pre-trained word vector which can be downloaded from here (https://wikipedia2vec.github.io/wikipedia2vec/pretrained/)
    intent_classification\classification_data\enwiki_20180420_100d.txt
    
    (4).Intent recognition
    
        a. Classification model 1, where feedforward-network is used for intent recognition

        Training code：intent_classification\pytorch\feedforward_network\train.ipynb
        
        Prediction code：intent_classification\pytorch\feedforward_network\predict.ipynb
        

        b. Classification model 2, where textcnn is used for intent recognition

        Training code：intent_classification\pytorch\textcnn\train.ipynb
        
        Prediction code：intent_classification\pytorch\textcnn\predict.ipynb

        c. Classification model 3, where textrnn is used for intent recognition

        Training code：intent_classification\pytorch\textrnn\train.ipynb
        
        Prediction code：intent_classification\pytorch\textrnn\predict.ipynb

        d. Classification model 4, where textrcnn is used for intent recognition

        Training code：intent_classification\pytorch\textrcnn\train.ipynb
        
        Prediction code：intent_classification\pytorch\textrcnn\predict.ipynb
    
    5. After the intent is recognized, Convert the recognized intent and the extracted slot (ie the recognized entity) into cypher language, and query in neo4j to get the answer
         Use the classification model to predict the intent category of the user's question, convert different intent categories into different cypher languages, and get answers from neo4j queries.
![image](https://raw.githubusercontent.com/tadiyoo/Satellite_KnowledgeGraph/master/img/qa.png)