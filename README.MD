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
