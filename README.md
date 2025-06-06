# Neo4j Open Exercise (Modeling, Cypher Queries, and Going Further)

### Introduction

The aim of this exercise is to explore an example that is not so far from our daily journey at ACC by taking the example of the **LAKANO** company. This company, based in Lacanau, manufactures bodyboards (see figure 1) and is asking its engineering department to figure out a traceability solution for them to be able to track their products.

<center>
    <div style="display:flex;flex-direction:column;padding:0 20%">
        <img src='bodyboard.png' />
        <span>Fig1: Bodyboard Schema </span>
    </div>
</center>

First, they want to focus on two types of traceability:

- _genetic traceability_: which products are used to produce another one
- _machine traceability_: which machines were used to produce a given product.

### Simplification

#### Products Manufactured

The LAKANO company produces two different products: the `classic` and the `performance` bodyboards. The difference between both is that the performance one has stringers to make it more robust and allow the surfer to take higher waves.

#### Recipe / Product Structure

We are simplifying to represent a bodyboard easily as a graph. Here is our product recipe:

```txt
-- bodyboard (1x)
---- nose (1x)
---- skin (1x)
---- sides (2x)
---- core (1x)
------ soft foam (1x)
------ hard foam (1x)
------ glue (1x)
------ stringer (0x or 2x)
```

#### Technical Document

Here you will find different table extracts coming from the product lifecycle management tool of the company LAKANO.

##### Bill of Materials (representing product recipes):

###### Performance Bodyboard

| Component Name | Component Code | Type         |
| -------------- | -------------- | ------------ |
| Bodyboard      | PBD            | Manufactured |
| Skin           | PSK            | Purchased    |
| Side           | PSD            | Purchased    |
| Core           | PCR            | Manufactured |
| Soft Foam      | PSF            | Purchased    |
| Hard Foam      | PHF            | Purchased    |
| Glue           | PGL            | Purchased    |
| Stringer       | PST            | Purchased    |

###### Classic Bodyboard

| Component Name | Component Code | Type         |
| -------------- | -------------- | ------------ |
| Bodyboard      | CBD            | Manufactured |
| Skin           | CSK            | Purchased    |
| Side           | CSD            | Purchased    |
| Core           | CCR            | Manufactured |
| Soft Foam      | CSF            | Purchased    |
| Hard Foam      | CHF            | Purchased    |
| Glue           | CGL            | Purchased    |

##### Bill of Equipment (Representing the factory and machines and their location, here the same machines are used for Classic and Performance Bodyboards, the `X` represents the machine ID increment (1, 2, 3, or 4) as there could be multiple machines of the same type)

| Machine Name/Function | Machine Code | Number |
| --------------------- | ------------ | ------ |
| Bodyboard Assembly    | MBAX         | 4      |
| Core Assembly         | MCAX         | 4      |

##### Unique ID nomenclature of an item

Each product (final and intermediate) has a unique product ID following this nomenclature: `{product_code}{machine_code}{date}{increment}` with the date having the following format `YYYYMMDD` and increment going from 001 to 999 as the production per product type per machine will never exceed 999. Note: for purchased items, the machine code will be replaced by XXXX and the date will be the date of arrival in the warehouse.

An example for the first Performance bodyboard produced of the year on machine MBA1 will have the following traceability ID `CBDMBA120240101001`.

### Questions

#### 1. Modeling

For these questions, a suggestion will be to use the web app [Arrows](https://arrows.app/).

1. Your first exercise will be to model a generic graph representing product nodes with their product relationships.
   
   <center>
        <div style="display:flex;flex-direction:column;padding:0 20%">
            <img src='LAKANO MODEL_1.png' />
            <p><i> - I created only one node Bodyboard, but i could have created both Classic bodyboard and Performance bodyboard, either, i could have created a "Type" property on this node, but with the code CBD and PBD it is already easy to retrieve the kind of bodyboard.</i></p>
            <p><i>- The bodyboard components (Nose, Sides, Stringer, Skin) are linked to the node Bodyboard, and the node "Core" is also considered as a component. The Glue, Soft Foam and Hard Foam are linked to the core node as it is used to make it.</i></p>
    <p><i>- The property "Manufactured" added to the components node is a Boolean, so we can distinguish what is manufactred or not.</i></p>
        </div>
    </center>

2. Now please add the machine nodes and edges to this representation.

   <center>
        <div style="display:flex;flex-direction:column;padding:0 20%">
            <img src='LAKANO_MODEL_2.png' />
        </div>
    </center>
    <p><i>I created two top level nodes MBA & MCA (with label machines) linked to the bodyboard node and core node, so it is easier to read and for maintenance</i></p>
    
3. Finally, we now want to extend the graph to purchased items supplier traceability. For that, we know that Core Purchased items are supplied by the company `CORETECH` and Bodyboard Purchased items are supplied by the company `SKIN&SKIM`. Add supplier nodes and edges to your representation.

   <center>
        <div style="display:flex;flex-direction:column;padding:0 20%">
            <img src='LAKANO_MODEL_3.png' />
        </div>
    </center>
    <p><i> Added the two suppliers nodes as they supplies two different kind of components, same reason as for the machines.</i></p>

#### 2. Queries

In this part, if you could generate some mock data to a local or online Neo4j instance, that will help you to query your graph to validate your requests. Nevertheless, this generation is optional.

```txt
You can check the instance i created on auraDB

uri : neo4j+s://3ab36c63.databases.neo4j.io
user : neo4j
password : wJCvkHjEnH5IesMdXBfbhB6qD0TuzUloWPIe_CE_pmY
```

All code parts from this section should be written in Cypher.

1. Write a query to retrieve all components of a given bodyboard.

```txt
MATCH (bodyboard:Bodyboard {uniqueID: 'PBDMBA120240101001'})-[:IS_MADE_OF]->(bodyboardCpts:Component)
OPTIONAL MATCH (bodyboardCpts:Component {name: 'Core'})-[:IS_MADE_OF]->(coreCpts:Component)
RETURN 
  bodyboard.uniqueID as BodyboardID,
  COLLECT(DISTINCT bodyboardCpts.name) as bodyboard_components, 
  COLLECT(DISTINCT coreCpts.name) as core_components
```

2. Write a query to count how many items have been produced using the `MBA1` machine. 

```txt
MATCH (item)-[:PRODUCED_BY]->(machine:Machine {code: 'MBA1'})
RETURN COUNT(item) AS items_produced_by_MBA1;
```

3. Write a trigger to set the date of production as a node property [Optional]. Then write the same query as question 2 but add "and during the month of April".

```txt
I can't install APOC plugin on my AuraDB instance, so i was not able to create the trigger, but we could imagine something like that :

CALL apoc.trigger.add(
  'setProductionDate',
  'UNWIND $createdBodyboard AS n
   SET n.productionDate = date()',
  {phase: 'after'}
);

Below, the query listing the items produced by machine MBA1 during the month of April :

MATCH (item)-[:PRODUCED_BY]->(machine:Machine {code: 'MBA1'})
WHERE item.productionDate >= date('2024-04-01') AND item.productionDate < date('2024-05-01')
RETURN COUNT(item) AS items_produced_by_MBA1_in_April;
```
4. Write a query to verify that all performance bodyboards are composed of two stringers.

```txt
In my instance, the PBD is composed of only one stringer. But the query should be this:

MATCH (bodyboard:Bodyboard {code: 'PBD'})-[:IS_MADE_OF]->(stringer:Component {name: 'Stringer'})
WITH bodyboard, COUNT(stringer) AS stringerCount
WHERE stringerCount <> 2
RETURN bodyboard.uniqueID AS BodyboardID, stringerCount;
```
5. We figured out that glue used was defective for some classic bodyboards. Here is the ID: `CGLXXXX20240101023`. We want you to output bodyboard IDs made from this glue.

```txt
MATCH (core:Component)-[:IS_MADE_OF]->(component:Component {uniqueID: 'CGLXXXX20240101023'})
MATCH (bodyboard:Bodyboard)-[:IS_MADE_OF]->(core)
RETURN bodyboard.uniqueID AS BodyboardID;
```

#### 3. Going further

1. Imagine that the LAKANO company is going global and opens two other factories in Nazare (Portugal) and Tahiti (France). How can you tune the graph to be able to track the factory location?

```txt
I didn't implement the factories into AuraDB instance, but we could tune the graph this way :
```
<center>
        <div style="display:flex;flex-direction:column;padding:0 20%">
            <img src='LAKANO_MODEL_4.png' />
        </div>
</center>

```txt
Then, we could find all bodyboards produced by machines located in a specific factory, by doing this :

MATCH (bodyboard:Bodyboard)-[:PRODUCED_BY]->(machine:Machine)-[:LOCATED_IN]->(factory:Factory {name: 'Nazare Factory'})
RETURN bodyboard.uniqueID AS BodyboardID;
```
2. We're producing more and more and so are facing issues when searching inside the graph. What could be solutions to optimize the search efficiency?

```txt
There are many solutions..
- The first one and the easiest way is to create indexes on the most frequently searched properties:
CREATE INDEX FOR (n:Bodyboard) ON (n.uniqueID);
CREATE INDEX FOR (n:Component) ON (n.name);
CREATE INDEX FOR (n:Machine) ON (n.uniqueid);

- We could use Pagination with SKIP and LIMIT

- Using labelling strategies to reduce the number of nodes neo4j needs to traverse

- Batch Processing using CALL apoc.periodic.iterate (if APOC is availablet) to process large datasets in smaller, manageable batches.

... look at hardware side ?
```

3. The quality team is asking to have easy access to production properties. For example, when the core is assembled, the glue is heated at a given temperature. The max of this temperature is saved. What could be your solution to embed such properties into the graph and what could be a possible query to retrieve the core glue temperature of a given module?

```txt
First, we could extend the Graph Model with Production Properties by attaching the maxTemperature property to the IS_MADE_OF relationship between the Core and Glue.

MATCH (core:Component {name: 'Core'})-[r:IS_MADE_OF]->(glue:Component {name: 'Glue'})
SET r.maxTemperature = 180  // Example value

Then the query to retrieve the maxTemperature should be this :

MATCH (bodyboard:Bodyboard {uniqueID: 'PBDMBA120240409001'})-[:IS_MADE_OF]->(core:Component {name: 'Core'})-[r:IS_MADE_OF]->(glue:Component {name: 'Glue'})
RETURN r.maxTemperature AS MaxGlueTemperature;
```
