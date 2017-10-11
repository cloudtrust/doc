# CockroachDB

In this chapter we will discuss the methods with which support for CockroachDB was added to Keycloak.

## Database schema creation

Keycloak uses Liquibase to create the database schema and all constants (see [Creating the database](create_database.md)).  Liquibase supports PostgreSQL, but not directly CockroachDB. However, since CockroachDB uses the PostgreSQL jdbc driver and supports the syntax, Liquibase is mostly compable with CockroachDB.

However, there are quite a few notable exceptions to this compatibility, and they arise when Keycloak's "normal" database changelogs are used. To correct the problem, a new set of changelogs specifically for CockroachDB are generated, mostely using the `ChangeLogEditor` - a java tool developped specifcally for this purpose.

The following subsections list the incompatibilities between the Keycloak changelogs and CockroachDB, and the modifications made.

### Adding primary keys

In CockroachDB primary keys must be created with the tables and cannot be manipulated afterwards, but in the Keycloak changelogs the operation is always seperate. To correct this, the syntax is modified to group the creation of primary keys in the create table instruction.

However, in a few cases (about 8), "primary columns" are removed, and a new key is added. Since the table generation is in another file, in this case:

1. A new table is created with the correct structure
2. The contents are copied from the old table to the new table
3. The old table is dropped
4. The new table is renamed to the correct table name. 

### Adding foreign keys

Foreign keys can be added and removed without problem in CockroachDB, however the column(s) MUST have a index defined on it. The solution creates indexes on all columns in a new changelog (to avoid precedence problems), and moves all "add foreign keys" to a third column (for the same reason.

### Removing unique constraints

In CockroachDB altering a table to add a unique constraint in fact creates a unique index, and adds an entry to the list of constraints. And in fact creating a unique index does exactly the same thing, meaning that the two are interchangable. However, while it is possible to drop a unique index (which also drops the contraint), the CockRoachDB does not allow the syntax to drop the constraint. 

The modification changes all instances of "drop unique constraint" to "drop uinique index".

### Activities that must or should be done

This TODO list can be editied as the following tasks are resolved:

* Only create indexes for foreign keys when one does not already exist
* Remove indexes created for foreign keys when they are removed
* Push a modification to CockroachDB to ensure that "remove unique contraint" acts like "remove unique index"
    