# Database paradigms: relational, document and graph

One problem modelled three ways, so the differences between the paradigms show up as
something other than a definition. Coursework, completed by hand.

The write-up that goes with this is a 19-page document covering each implementation with the
queries and their results.

## What is here

```
oracle/schema-and-procedures.sql    Oracle export: sequences, tables, a view, a procedure
mongodb/library.*.json              five exported collections
neo4j/neo4j-*.dump                  a Neo4j database dump
```

### Oracle (relational)

A campus registration schema: `DEPARTMENT`, `INSTRUCTOR`, `MODULE`, `STUDENT` and a
`REGISTRATION` join table with a composite primary key, under 17 constraints and five
sequences for surrogate keys. `INSTRUCTOR_MODULES_VIEW` joins instructors to the modules they
teach.

The part worth reading is `REGISTER_STUDENT`. It does not trust its arguments: it checks that
the student exists, that the module exists, and that the pair is not already registered,
returning early on each. Only then does it insert and commit, and a `WHEN OTHERS` handler
rolls back if anything in between fails.

### MongoDB (document)

A library: `author`, `book`, `member`, `bookauthor` and `borrowedbooks`. The same data that
needs a join table in Oracle is held as documents here, and the write-up shows the
`$lookup` aggregation that does what the SQL `JOIN` does.

### Neo4j (graph)

The same library as a graph. Members, books and authors are nodes; `BORROWED` and
`IS_WRITTEN_BY` are relationships, and `BORROWED` carries the borrow date, due date, return
date and fine on the relationship itself rather than in a separate table. The Cypher in the
write-up walks the graph to produce the same member-borrowing table that the `$lookup` and
the SQL `JOIN` produce.

## Known issues, found by reading it back

Not fixed, because the point of keeping them is that they are honest about where this was
written and what it does.

1. **`REGISTER_STUDENT` swallows its errors.** The `WHEN OTHERS` handler rolls back, which is
   right, and then prints to `DBMS_OUTPUT` and returns normally, which is not. A caller has
   no way to tell a successful registration from a failed one. `RAISE_APPLICATION_ERROR`
   after the rollback would fix it.
2. **The validation failures are the same.** A missing student number prints a message and
   returns, so the procedure reports success to anything that is not watching the console.
3. **`WHEN OTHERS` with no re-raise catches everything**, including the errors that mean the
   database itself is in trouble, and reports them the same way as a typo in a module code.
4. **The export carries an earlier exercise.** `CUSTOMER`, `ORDERS` and `PRODUCT` sit in the
   same schema with their sequences, left in because that is what the schema actually
   contained. They are not part of the registration model and have no rows.
5. **Five rows per table.** Enough to show the relationships resolve and nothing like enough
   to say anything about how any of this behaves at size.

## Running it

The Oracle file is a schema export, so it expects to be run as a user that can create objects
and it qualifies everything with the owner `LIBRARYDB`. The Mongo files import with
`mongoimport` one collection at a time. The Neo4j file is a dump, restored with
`neo4j-admin database load`, not a script.

## A note on the data and the names

Every row is invented: Harry Potter, 1984, John Doe, Jane Roe. The schema owner in the Oracle
export was renamed from the account it was written under, and the Mongo collections were
renamed off the institution's database name. Nothing was changed inside the data.
