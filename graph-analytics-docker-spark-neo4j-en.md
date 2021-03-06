# A Docker Image for Graph Analytics on Neo4j with Apache Spark GraphX

---

Author: Kenny Bastani

---

I've just released a useful new [Docker image for graph analytics](https://registry.hub.docker.com/u/kbastani/neo4j-graph-analytics/) on a [Neo4jgraph database](http://www.neo4j.com/) with Apache Spark GraphX. This image deploys a container with [Apache Spark](https://spark.apache.org/) and uses [GraphX](https://spark.apache.org/graphx/) to perform ETL graph analysis on subgraphs exported from Neo4j. This docker image is a great addition to Neo4j if you're looking to do easy PageRank or community detection on your graph data. Additionally, the results of the graph analysis are applied back to Neo4j.

This gives you the ability to optimize your recommendation-based Cypher queries by filtering and sorting on the results of the analysis.

![alt](http://resource.docker.cn/tables-and-graphs.png)

*Photo credit AMPLab Berkley*

As an example, if I wanted to calculate the popularity of my Twitter network, I could run a PageRank and Triangle Count analysis on my community subgraph and calculate the popularity of users using Cypher.

![alt](http://resource.docker.cn/snap1.png)

## Supported Algorithms

There are four supported graph analysis algorithms available in this version of the image.

- *PageRank*

- *Triangle Counting*

- *Connected Components*

- *Strongly Connected Components*

Each of these algorithms come default with Spark GraphX.

### Extending the Neo4j Server

The Neo4j server is extended using an [unmanaged extension](http://neo4j.com/docs/stable/server-unmanaged-extensions.html) that adds a REST API endpoint to Neo4j for submitting graph analysis jobs to Apache Spark GraphX. The results of the analysis are applied back to the nodes in Neo4j as property values, making the results queryable using Cypher.

## Installation/Deployment

Installation requires 3 docker image deployments, each containing a separate linked component.

- Hadoop HDFS (sequenceiq/hadoop-docker:2.4.1)
- Neo4j Graph Database (kbastani/docker-neo4j:latest)
- Apache Spark Service (kbastani/neo4j-graph-analytics:latest)

Pull the following docker images:

```
docker pull sequenceiq/hadoop-docker:2.4.1
docker pull kbastani/docker-neo4j:latest
docker pull kbastani/neo4j-graph-analytics:latest
```

After each image has been downloaded to your Docker server, run the following commands in order to create the linked containers.

```
# Create HDFS
docker run -i -t --name hdfs sequenceiq/hadoop-docker:2.4.1 /etc/bootstrap.sh -bash

# Create Mazerunner Apache Spark Service
docker run -i -t --name mazerunner --link hdfs:hdfs kbastani/neo4j-graph-analytics

# Create Neo4j database with links to HDFS and Mazerunner
# Replace <user> and <neo4j-path>
# with the location to your existing Neo4j database store directory
docker run -d -P -v /Users/<user>/<neo4j-path>/data:/opt/data --name graphdb --link mazerunner:mazerunner --link hdfs:hdfs kbastani/docker-neo4j
```

### Use Your Existing Neo4j Database

To use an existing Neo4j database, make sure that the database store directory, typically `data/graph.db`, is available on your host OS. Read the [setup guide](https://github.com/kbastani/docker-neo4j#start-neo4j-container) for kbastani/docker-neo4j for additional details.

### Accessing the Neo4j Browser

The Neo4j browser is exposed as a container on port 7474. If you're wanting to test this deployment on your development machine and are using [boot2docker](http://boot2docker.io/) on MacOSX, follow the directions [here](https://github.com/kbastani/docker-neo4j#boot2docker) to access the Neo4j browser on your environment.

## Usage Directions

Graph analysis jobs are started by accessing the following endpoint:

```
http://localhost:7474/service/mazerunner/analysis/{analysis}/{relationship_type}
```

To choose an analysis to start running, replace `{analysis}` in the URL above with one of the following keys:

- pagerank
- triangle_count
- connected_components
- strongly_connected_components

To select a subgraph from Neo4j that you would like to analyze, replace the `{relationship_type}` in the URL above with a relationship type in your Neo4j database. The nodes that are connected by that relationship will form the graph that will be analyzed. For example, the equivalent Cypher query would be the following:

```
MATCH (a)-[:FOLLOWS]->(b)
RETURN id(a) as src, id(b) as dst
```

Just to illustrate, if you ran a `pagerank` analysis on the `FOLLOWS` relationship type, the following Cypher query will display the results:

```
MATCH (a)-[:FOLLOWS]-()
RETURN DISTINCT id(a) as id, a.pagerank as pagerank
ORDER BY pagerank DESC
```

## Usage Examples

To run graph analysis algorithms, HTTP GET request on the following Neo4j server endpoints:

### PageRank

```
http://172.17.0.21:7474/service/mazerunner/analysis/pagerank/FOLLOWS
```

- Gets all nodes connected by the `FOLLOWS` relationship and updates each node with the property key `pagerank`.

- The value of the `pagerank` property is a float data type, ex. `pagerank: 3.14159265359`.

- PageRank is used to find the relative importance of a node within a set of connected nodes.

### Triangle Counting

```
http://172.17.0.21:7474/service/mazerunner/analysis/triangle_count/FOLLOWS
```

- Gets all nodes connected by the `FOLLOWS` relationship and updates each node with the property key `triangle_count`.

- The value of the `triangle_count` property is an integer data type, ex. `triangle_count: 2`.

- The value of `triangle_count` represents the count of the triangles that a node is connected to.

- A node is part of a triangle when it has two adjacent nodes with a relationship between them. The `triangle_count` property provides a measure of clustering for each node.

### Connected Components

```
http://172.17.0.21:7474/service/mazerunner/analysis/connected_components/FOLLOWS
```

- Gets all nodes connected by the `FOLLOWS` relationship and updates each node with the property key `connected_components`.

- The value of `connected_components` property is an integer data type, ex. `connected_components: 181`.

- The value of `connected_components` represents the *Neo4j internal node ID* that has the lowest integer value for a set of connected nodes.

- Connected components are used to find isolated clusters, that is, a group of nodes that can reach every other node in the group through a *bidirectional traversal*.

### Strongly Connected Components

```
http://172.17.0.21:7474/service/mazerunner/analysis/strongly_connected_components/FOLLOWS
```

- Gets all nodes connected by the `FOLLOWS` relationship and updates each node with the property key `strongly_connected_components`.

- The value of `strongly_connected_components` property is an integer data type, ex. `strongly_connected_components: 26`.

- The value of `strongly_connected_components` represents the *Neo4j internal node ID* that has the lowest integer value for a set of strongly connected nodes.

- Strongly connected components are used to find clusters, that is, a group of nodes that can reach every other node in the group through a directed traversal.

## Getting started

To get started, head over to the [neo4j-graph-analytics Docker Repository](https://registry.hub.docker.com/u/kbastani/neo4j-graph-analytics/) and follow the directions to get started. If you're new to Docker and looking to get your feet wet, take a look at this great online tutorial: [https://www.docker.com/tryit/](https://www.docker.com/tryit/)

If you have any questions or need help getting setup, please feel free to comment below. Also, comment your ideas for additional graph analytics algorithms that could be added to this library.

Have fun!

---

Original source: [A Docker Image for Graph Analytics on Neo4j with Apache Spark GraphX](http://www.kennybastani.com/2014/11/graph-analytics-docker-spark-neo4j.html)

