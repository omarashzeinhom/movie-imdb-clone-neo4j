# Imdb Arabic Clone in Go

## Run By

```go
//run by
go run ./cmd/neoflix
```

```json
//https://graphacademy.neo4j.com/courses/app-go/0-setup/2-sandbox/
//By default, a Neo4j sandbox instance exists for 3 days. You can extend it for another 7 days by going to the sandbox site and extending it in the details (right-most down arrow) for the recommendations sandbox.
{
  "APP_PORT": 3000,
  "NEO4J_URI": "bolt://44.200.34.147:7687",
  "NEO4J_USERNAME": "neo4j",
  "NEO4J_PASSWORD": "batch-numerals-distress",
  "JWT_SECRET": "secret",
  "SALT_ROUNDS": 10
}

```

- Note: This is not sql ! its GraphQueryLanguage in Cypher ,
Unfortunatenly there is no code fence extesnison for it on git hub

```sql
/*CYPHER MAIN QUERIES */

MATCH (p:Person) /* GETS PROPERTY */
WHERE p.name ='Kevin Bacon' /*FILTERS PROPERTY*/
RETURN p.born
/*p.born Returns 1958*/
```


### References 

1. [Adding GraphQL syntactical highlighting for code blocks](https://meta.stackexchange.com/questions/358925/adding-graphql-syntactical-highlighting-for-code-blocks)
2. [Handling Driver Errors](https://graphacademy.neo4j.com/courses/app-go/2-interacting/6-driver-errors/)
```

result, err := session.Run(
	"MTCH (n) RETURN x(n)", nil)

/**
	Neo4jError: Neo.ClientError.Statement.SyntaxError (Invalid input 'T': expected 'a/A' or 'e/E' (line 1, column 2 (offset: 1))
	"MTCH (n) RETURN x(n)"
  	 ^)]
*/

// Cast error as a Neo4j Error
neo4jError := err.(*neo4j.Neo4jError)

fmt.Println(neo4jError.Code) // <1> Neo.ClientError.Statement.SyntaxError
fmt.Println(neo4jError.Msg)  // <2> (Invalid input 'T':...

// The error code can be further broken down into the following parts:
fmt.Println(neo4jError.Classification()) // ClientError
fmt.Println(neo4jError.Category())       // Statement
fmt.Println(neo4jError.Title())          // SyntaxError 
```

```
//ERROR CODE FORMAT , includes four parts 
Neo.[Classification].[Category].[Title]
(1)        (2)          (3)       (4)
```


```


    Every Neo4j Error code is prefixed with Neo.

    The Classification provides a high-level classification of the error - for example, a client-side error or an error with the database.

    The Category provides a higher-level category for the error - for example, a problem with clustering, a procedure or the database schema.

    The Title provides specific information about the error that has occurred.

```

```
Common Exception Types

Depending on the nature of the error, you may receive one of the following exceptions:

    neo4j.exceptions.Neo4jError - Raised when the Cypher engine returns an error to the client.

    neo4j.exceptions.ClientError - The Client sent a bad request - changing the request might yield a successful outcome.

        neo4j.exceptions.CypherSyntaxError - Raised when the Cypher statement contains one or more syntax errors

        neo4j.exceptions.CypherTypeError - Raised when or more of the data types in the query is incorrect

        neo4j.exceptions.ConstraintError - Raised when action is rejected due to a constraint violation

        neo4j.exceptions.AuthError - Raised when authentication failure occurs.

        neo4j.exceptions.Forbidden - Raised when the action is forbidden for the authenticated user

    neo4j.exceptions.TransientError - The database cannot service the request right now, retrying later might yield a successful outcome

        neo4j.exceptions.ForbiddenOnReadOnlyDatabase - The write cypher you are requesting cannot be run on a readonly database

        neo4j.exceptions.NotALeader - The write query cannot be executed on the current server because it is not the leader of the 
```
`For a comprehensive list of status codes, see Status Codes in the Neo4j Documentation.`[Status Codes - Neo4j](https://neo4j.com/docs/status-codes/current/)
