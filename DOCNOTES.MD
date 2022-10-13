[Sessions and Transactions - Neo4j](https://graphacademy.neo4j.com/courses/app-go/2-interacting/1-transactions/)

A session is a container for a sequence of transactions. Sessions borrow connections from a pool as required and are considered lightweight and disposable.

It is important to remember that sessions are not the same as database connections. When the Driver connects to the database, it opens up multiple TCP connections that can be borrowed by the session. A query may be sent over multiple connections, and results may be received by the driver over multiple connections.

Instead, sessions should be considered a client-side abstraction for grouping units of work, which also handle the underlying connections. The connections themselves are managed internally by the driver and are not directly exposed to the application.

To open a new session, call the NewSession() method on the driver.

Open a new Session

```go
session := driver.NewSession(neo4j.SessionConfig{})
defer session.Close()
```

Auto-commit Transactions

Auto-commit transactions are a single unit of work that are immediately executed against the DBMS and acknowledged immediately. You can run an auto-commit transaction by calling the Run() method on the session object, passing in a Cypher statement as a string and optionally an object containing a set of parameters.
go
Auto-commit Transactions

```go
result, err := session.Run(
 "MATCH (p:Person {name: $name}) RETURN p",
 map[string]interface{}{"name": "Tom Hanks"})
```

Read Transactions

When you intend to read data from Neo4j, you should execute a Read Transaction. In a clustered environment (including Neo4j Aura), read queries are distributed across the cluster.

The session provides a ReadTransaction() function, which expects a single parameter, a function that represents the unit of work. The first argument passed to the function will be a transaction object, on which you can call the Run() function to execute a Cypher statement. As with the session.Run example above, the first argument for the Run() function should be a Cypher statement, and any parameters in the Cypher statement should be passed as named parameters

```go
result, err := session.ReadTransaction(func(transaction neo4j.Transaction) (interface{}, error) {
 result, err := transaction.Run(
  "MATCH (n) RETURN count(n) AS count", map[string]interface{}{})
 if err != nil {
  return nil, err
 }
 record, err := result.Single()
 if err != nil {
  return nil, err
 }
 count, _ := record.Get("count")
 return count.(int64), result.Err()
})
```

```go
rawName, err := session.WriteTransaction(func(transaction neo4j.Transaction) (interface{}, error) {
 result, err := transaction.Run(
  "CREATE (p:Person {name: $name}) RETURN p",
  map[string]interface{}{"name": name})
 if err != nil {
  return nil, err
 }

 personRecord, err := result.Single()
 if err != nil {
  return nil, err
 }
 personNode, _ := personRecord.Get("p")
 person := personNode.(neo4j.Node)

 return person.Props["name"], result.Err()
})
if err != nil {
 return "", err
}
```

Creating an Manual Transaction

```
// Begin Transaction
tx, err := session.BeginTransaction()
```

Commit and Rollback

```go
/ Begin Transaction
tx, err := session.BeginTransaction()
if err != nil {
 return "", err
}

// Run a Cypher Query
result, err := tx.Run(cypher, params)

// If something goes wrong then roll back the transaction
if err != nil {
 if rollbackErr := tx.Rollback(); rollbackErr != nil {
  // Go 1.13 %w feature to wrap errors
  return "", fmt.Errorf("rollback error (%v) happened after %w", rollbackErr, err)
 }
 return "", err
}

// Otherwise, commit the transaction
if err = tx.Commit(); err != nil {
 return "", err
}
```

go
Closing a Session

```go
defer session.Close()
```

Processing Results

Query results are typically consumed as a stream of records. The drivers provide a way to iterate through that stream.
Result

Here is an example query which retrieves a list of :Person nodes related to a given Movie.

```go
func getActors() (string, error) {
 // <1> Initiate Driver
 driver, err := neo4j.NewDriver("neo4j://localhost:7687",
  neo4j.BasicAuth("neo4j", "letmein", ""))

 // <2> Check for driver instantiation error
 if err != nil {
  return "", err
 }

 // <3> Defer closing of the driver
 defer driver.Close()

 // <4> Create a new Session
 session := driver.NewSession(neo4j.SessionConfig{DatabaseName: "movies", AccessMode: neo4j.AccessModeWrite})

 // <5> Defer closing the session
 defer session.Close()

 // <6> Execute Cypher and get Result
 result, queryErr := session.Run(
  "MATCH (p:Person)-[r:ACTED_IN]->(m:Movie {title: $title}) RETURN p, r, m",
  map[string]interface{}{"title": "Arthur"})

 // <7> Handle Query Errors
 if queryErr != nil {
  // Problem executing the query, maybe a syntax error?
  return "", queryErr
 }

 // <8> For each Record in the Result
 for result.Next() {
  // <9> Get the next record
  record := result.Record()

  // <10> Interact with the record object
  fmt.Println(record.Keys) // ['p', 'r', 'm']
  // Access a value by its index
  fmt.Println(record.Values[0].(neo4j.Node)) // The Person node
  // Access a value by its alias
  movie, _ := record.Get("movie")
  movieNode := movie.(neo4j.Node)
  fmt.Println(movieNode) // The Movie node
 }

 return "", nil
}
```

A new instance of the Driver is created

1. If an error occurs when creating the driver instance, it can be handled by checking the error returned by the declaration
2. Once the function returns, close the driver instance and release any resources
3. Create a new session with the intention of querying the movies database
4. Close the session once the function returns
5. Execute a Cypher statement and get the result
6. If any errors occur during the execution of the query, for example connection errors, timeouts or syntax errors
7. Use a for loop to iterate through the stream of results. As long as Next() returns true, there are still records remaining in the stream
8. Consume the next result from the stream
9. Interact with the record

Accessing a Record

```go
// .Next() returns false upon error
for result.Next() {
    record := result.Record()
    handleRecord(record)
}
// Err returns the error that caused Next to return false
if err = result.Err(); err != nil {
    handleError(err)
}
```

NextRecord()

Alternatively, you can use the NextRecord() method which has the same outcome with less code.

```go
for result.NextRecord(&record) {
    fmf.Println(record.Keys)
}
```

Getting the only result with Single()

```go
// Get the first and only result from the stream.
first, err := record.Single()
```

Get all remaining results with Collect() The Collect() Method

```go
remaining, remainingErr := result.Collect()
```

Consume remaining results with Consume()

Calling the Consume() method will discard any remaining records from the stream and return a ResultSummary object which holds information about the statement execution, including database information, execution time and in the case of a write query, statistics on changes made to the database as a result of the statement execution.

```go

summary := result.Consume()

// Time in milliseconds before receiving the first result
fmt.Println(summary.ResultAvailableAfter())

// Time in milliseconds once the final result was consumed
fmt.Println(summary.ResultConsumedAfter())
```

Exploring Records

The record variable above provides an interface to retrieve individual values. Take the example above, each record will contain information on :Movie and :Person nodes, and the :ACTED_IN relationship.

To access the keys available in each record, you can use the .Keys property. This is an array of strings.

```go
fmt.Println(record.Keys) // ['p', 'r', 'm']
```

To access these values, we can either use the index of the key above, or the alias specified in the RETURN portion of the Cypher statement.

```go
// Access a value by its index
fmt.Println(record.Values[0].(neo4j.Node)) // The Person node
// Access a value by its alias
movie, _ := record.Get("movie")
movieNode := movie.(neo4j.Node)
fmt.Println(movieNode) // The Movie node
```

Nodes & Relationships

Working with Nodes and Relationships are very similar. Both have an .Id property which represents the internal Neo4j ID as an int64 and a .Props property which holds a map of properties.

As an example, let’s take the following code snippet:

```go
result, err := transaction.Run(
 "MATCH path = (p:Person)-[r:ACTED_IN]->(m:Movie {title: $title}) RETURN p, r, m, path",
 map[string]interface{}{"title": "Arthur"})

if err != nil {
 return nil, err
}

```

Extracting Values

As mentioned in the previous lesson, you can use the alias as defined in the RETURN portion of the Cypher statement:

```go
for result.NextRecord(&record) {
 // Get the value of `m`
 movie, _ := record.Get("m")
 movieByAlias := movie.(neo4j.Node) // m, a :Movie node as type neo4j.Node()
}
```

Or alternatively, you can either retrieve a value by its index in the Values property:

```go
// Check keys to find the index
fmt.Println(result.Keys()) // ['p', 'm', 'a', 'path']

for result.NextRecord(&record) {
 // Get the first value, in this case `p`
 personByIndex := record.Values[0].(neo4j.Node) // p, a :Person node as type neo4j.Node()
}
```

Neo4j.Node is a struct that allows you to access:

    .Id - The Internal ID of the Node as an int64
    eg 8491

    .Labels - An array of strings representing the labels attributed to the Node
    eg. ['Person, 'Actor']

    .Props - A map of properties assigned to the Node
    eg. {name: 'Tom Hanks', tmdbId: '31' }

```
personNode, _ := record.Get("p")
person := personNode.(neo4j.Node)

fmt.Println(person.Id)     // (1)
fmt.Println(person.Labels) // (2)
fmt.Println(person.Props)  // (3)
```

Internal IDs
Internal IDs refer to the position in the Neo4j store files where the record is held. These numbers can be re-used, a best practice is to always look up a node by an indexed property rather than relying on an internal ID.

Working with Relationships

```go
type Relationship struct {
 Id      int64                  // Identity of this Relationship.
 StartId int64                  // Identity of the start node of this Relationship.
 EndId   int64                  // Identity of the end node of this Relationship.
 Type    string                 // Type of this Relationship.
 Props   map[string]interface{} // Properties of this Relationship.
}
```

neo4j.Relationship is a struct that allows you to access:

    .Id - The internal ID of the relationship as an int64
    eg. 9876

    .Type - The type of the relationship
    eg. ACTED_IN

    .Props - A map of properties assigned to the Relationship
    eg. {role: 'Woody'}

    .StartId - The internal ID for the node at the start of the relationship

    .EndId - The internal ID for the node at the end of the relationship

```go
type Path struct {
 Nodes         []Node // All the nodes in the path.
 Relationships []Relationship
}
```

If you return a path of nodes and relationships, they will be returned as an instance of a neo4j.Path.

```go
returnedPath, _ := record.Get("path")
path := returnedPath.(neo4j.Path)

nodes := path.Nodes
relationships := path.Relationships

for node := range nodes {
 fmt.Println(node) // neo4j.Node
}
for relationship := range relationships {
 fmt.Println(relationship) // neo4j.Relationship
}

```

Path Segments

A path is split into segments representing each relationship in the path. For example, say we have a path of (p:Person)-[:ACTED_IN]→(m:Movie)-[:IN_GENRE]→(g:Genre), there would be two relationships.

    (p:Person)-[:ACTED_IN]→(m:Movie)

    (m:Movie)-[:IN_GENRE]→(g:Genre)

You can access the relationships within the path through the .Relationships property, and the nodes in the path can be accessed through the .Nodes property.

```go
returnedPath, _ := record.Get("path")
path := returnedPath.(neo4j.Path)

nodes := path.Nodes
relationships := path.Relationships

for node := range nodes {
 fmt.Println(node) // neo4j.Node
}
for relationship := range relationships {
 fmt.Println(relationship) // neo4j.Relationship
}
```

Path Segments

A path is split into segments representing each relationship in the path. For example, say we have a path of (p:Person)-[:ACTED_IN]→(m:Movie)-[:IN_GENRE]→(g:Genre), there would be two relationships.

    (p:Person)-[:ACTED_IN]→(m:Movie)

    (m:Movie)-[:IN_GENRE]→(g:Genre)

You can access the relationships within the path through the .Relationships property, and the nodes in the path can be accessed through the .Nodes property.

```go
returnedPath, _ := record.Get("path")
path := returnedPath.(neo4j.Path)

nodes := path.Nodes
relationships := path.Relationships

for node := range nodes {
 fmt.Println(node) // neo4j.Node
}
for relationship := range relationships {
 fmt.Println(relationship) // neo4j.Relationship
}
```

Temporal data types are extensions of Go’s time.Time type.

You can access the individual parts of the Time struct using the appropriate method.

```go
timeProperty, _ := record.Get("time")
time := timeProperty.(neo4j.Time).Time()

fmt.Println(time.Year())  // 2022
fmt.Println(time.Month()) // January
fmt.Println(time.Day())   // 4

// For Time, DateTime,
fmt.Println(time.Day()) // 4
```

Durations

The neo4j.Duration type provides properties for accessing the Neo4j duration type. A Neo4j duration type contains the following data:

    Months - an int64

    Days - an int64

    Seconds - an int64

    Nanos - an int

```go

// duration('P1Y2M3DT12H34M56S')
// 1 year, 2 months, 3 days; 12 hours, 34 minutes, 56 seconds
durationProperty, _ := record.Get("duration")
duration := durationProperty.(neo4j.Duration)

fmt.Println(duration.Months)  // 14 (1 year, 2 months = 14 months)
fmt.Println(duration.Days)    // 3
fmt.Println(duration.Seconds) // 45296
fmt.Println(duration.Nanos)   // 987600000
```

Spatial Data Types

Cypher has built-in support for handling spatial values (Points), and the underlying database supports storing these point values as properties on nodes and relationships.
Points

Points can be stored in Neo4j as 2D points (x and y, or latitude and longitude) or 3D points (x, y and z, or latitude, longitude and height).

The Neo4j Go Driver provides two types to represent these types, Point2D and Point3D. Both of these types have .X and .Y properties along with a .SpatialRefId which is used to represent the ID of the coordinate reference system.

```
SpatialRefId  Description  Cypher Example

7203
 

Point2D in the cartesian space.
 

point({x: 10, y: 20})
```
