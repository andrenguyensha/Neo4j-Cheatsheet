## Neo4j-Cheatsheet Fundamentals

Store any kind of data using the following graph concepts:

* **Node**: Graph data records
* **Relationship**: Connect nodes (has direction and a type)
* **Property**: Stores data in key-value pair in nodes and relationships
* **Label**: Groups nodes and relationships (optional)

# Cypher

## Match

### Match node

```cypher
MATCH (n:Person)
WHERE n.name = "Andre"
RETURN (n);
```

* **MATCH** clause to specify a pattern of nodes and relationships
* **(n:Person)** a single node pattern with label 'Person' which will assign matches to the variable `n`
* **WHERE** clause to constrain the results
* **n.name = "Andre"** compares name property to the value "Andre"
* **RETURN** clause used to request particular results

Gets gets the id<5> and id<0> nodes and creates a `:KNOWS` relationship between them

### Match nodes and relationships

```cypher
MATCH (n:Person)-[:KNOWS]-(friends)
WHERE n.name = "Andre"
RETURN n, friends
```

* **MATCH** clause to describe the pattern from known Nodes to found Nodes
* **(n)** starts the pattern with a Person (qualified by WHERE)
* **-[:KNOWS]-** matches "KNOWS" relationships (in either direction)
* **(friends)** will be bound to Andre's friends

### Match labels

```cypher
MATCH (n:Person)
RETURN n;
```

or

```cypher
MATCH (n)
WHERE n:Person
RETURN n;
```

### Match multiple labels

`:Car` **OR** `:Person` labels

```cypher
MATCH (n)
WHERE n:Person OR n:Car
RETURN n;
```

`:Car` **AND** `:Person` labels

```cypher
MATCH (n)
WHERE n:Person:Car
RETURN n;
```

### Match same properties

```cypher
MATCH (a:Person)
WHERE a.from = "Sweden"
RETURN a;
```


### Viewing the graph

```
match (n:MyNode)-[r]->(m)
return n, r, m;
```

### Finding paths between specific nodes*: 

```
match p=(a)-[:TO*]-(c) 
where a.Name='H'  and c.Name='P' 
return p limit 1;

match p=(a)-[:TO*]-(c) where a.Name='H' and c.Name='P' 
return p order by length(p) asc limit 1;
```
### Finding the length between specific nodes: 

```
match p=(a)-[:TO*]-(c) 
where a.Name='H'  and c.Name='P' 
return length(p) limit 1;
```

### Finding a shortest path between specific nodes: 

```
match p=shortestPath((a)-[:TO*]-(c)) 
where a.Name='A'  and c.Name='P' 
return p, length(p) limit 1;
```
### All Shortest Paths: 

```
MATCH p = allShortestPaths((source)-[r:TO*]-(destination)) 
WHERE source.Name='A' AND destination.Name = 'P' 
RETURN EXTRACT(n IN NODES(p)| n.Name) AS Paths;
```
### All Shortest Paths with Path Conditions: 

```
MATCH p = allShortestPaths((source)-[r:TO*]->(destination)) 
WHERE source.Name='A' AND destination.Name = 'P' AND LENGTH(NODES(p)) > 5 
RETURN EXTRACT(n IN NODES(p)| n.Name) AS Paths,length(p);
```

### Diameter of the graph: 

```
match (n:MyNode), (m:MyNode) 
where n <> m 
with n, m 
match p=shortestPath((n)-[*]->(m))
return n.Name, m.Name, length(p)

order by length(p) desc limit 1;
```

### Extracting and computing with node and properties: 

```
match p=(a)-[:TO*]-(c) 

where a.Name='H'  and c.Name='P' 
return extract(n in nodes(p)|n.Name) as Nodes, length(p) as pathLength, 
reduce(s=0, e in relationships(p)| s + toInt(e.dist)) as pathDist limit 1;
```

### Dijkstra's algorithm for a specific target node:

```
MATCH (from: MyNode {Name:'A'}), (to: MyNode {Name:'P'}), 

path = shortestPath((from)-[:TO*]->(to))

WITH REDUCE(dist = 0, rel in rels(path) | dist + toInt(rel.dist)) AS distance, path

RETURN path, distance
```

### Dijkstra's algorithm SSSP:

```
MATCH (from: MyNode {Name:'A'}), (to: MyNode), 

path = shortestPath((from)-[:TO*]->(to))

WITH REDUCE(dist = 0, rel in rels(path) | dist + toInt(rel.dist)) AS distance, path, from, to

RETURN from, to, path, distance order by distance desc
```

### Graph not containing a selected node: 

```
match (n)-[r:TO]->(m) 

where n.Name <> 'D' and m.Name <> 'D' 

return n, r, m
```

### Shortest path over a Graph not containing a selected node: 

```
match p=shortestPath((a {Name: 'A'})-[:TO*]-(b {Name: 'P'}))

where not('D' in (extract(n in nodes(p)|n.Name)))

return p, length(p)

```

### Graph not containing the immediate neighborhood of a specified node: 

```
match (d {Name:'D'})-[:TO]-(b)

with collect(distinct b.Name) as neighbors

match (n)-[r:TO]->(m)

where 

not (n.Name in (neighbors+'D')) 

and 

not (m.Name in (neighbors+'D')) 

return n, r, m

;

match (d {Name:'D'})-[:TO]-(b)-[:TO]->(leaf)

where not((leaf)-->())

return (leaf)

;

match (d {Name:'D'})-[:TO]-(b)<-[:TO]-(root)

where not((root)<--())

return (root)
```

### Graph not containing a selected neighborhood: 

```
match (a {Name: 'F'})-[:TO*..2]-(b) 

with collect(distinct b.Name) as MyList 

match (n)-[r:TO]->(m) 

where not(n.Name in MyList) and not (m.Name in MyList) 

return distinct n, r, m
```
