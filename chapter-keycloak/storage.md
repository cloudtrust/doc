# Keycloak Storage

This chapter contains the principals of  keycloak storage.

## Database Storage


### Creating the database structure in Keycloak

As long as an initial database is created, keycloak will be able to automatically create its initial state (tables, indexes, constants, etc...), without needing a administrator to run SQL scripts. It does this through the use of [Liquibase](http://www.liquibase.org/). The use of Liquibase also allows the database to be automatically modified when upgrading to a new version of Keycloak.

#### About Liquibase

Full documentation may be found on [Liquibase's website](http://www.liquibase.org/documentation/index.html "Liquibase documentation"), but the basic principal is the following: changes to the database are recorded within _Database change logs_, XML files that contain the instructions for manipulating the database. One can for example create tables, delete tables, maniuplate column, indexes, constraints, etc... The philosophy is to create a set of one or more of such database change logs for every new release, and therefore no matter what version you are at, Liquibase can update from your current status to the new one.

In order to know which files have been run, and therefore which files to run in the event of an upgade, liquibase creates itself the `databasechangelog` table which records which changelogs have run, and the 

An important note is that liquibase uses the jdbc driver to determine which database is actually being used when applying the commands in the XML files to the database. When databases reuse drivers from another database, which is for example the case for [cockroachdb](cockroachdb.md). However, this means that the transformation Liquibase does from XML &rarr; SQL may be imperfect. 

#### Use of Liquibase in Keycloak

In Keycloak all database changelog files can be found at `model/jpa/src/main/resources/META-INF`. The main file is `jpa-changelog-master.xml` which contains the list of all database change files to call, and the order in which they are called. Running Keycloak or its unit tests that access the DB will automatically run the master changelog, if necessary.

## Cache Storage

Keycloak uses Infinspan to provide an in-memory cache mechanism.

Sessions information are cached but they are not persisted in database. In a cluster of Keycloak (standalone-ha setup), the sesions informations are shared between instances to ensure HA.

All others entities (Users, Realm, ...) are cached and persisted in database. When an entity is created, edited or removed, the cache is updated accordingly.
In cluster of keyclaok (standalone-ha setup), an eviction list is synchronized between all instances but the value of the entity is not transmitted.

Cache behavior is configurable and can be enbled/disabled in the XML configuration file. 

## Transaction Management

Each call to the REST API are performed in one Session. SessionServletFilter ensure a begin of the transaction for each HTTP request to Keyclaok server. The Transaction is terminated by a commit or rollback executed by another servlet filter (TransactionCommitter)

Note that Job execution is also executed scope of a transaction. Transaction management for Jobs is ensured by runJobInTransaction methods in KeycloakModelUtils (and UserStorageManager.runJobIntransaction).

Transactions are managed by DefaultKeycloakTransactionManager. An implementation of KeycloakTransaction exists for each target storage (JpaKeycloakTransaction for the DB, InfinisanKeycloakTransaction for the caching) 



## CockroachDB

In this chapter we will discuss the methods with which support for CockroachDB was added to Keycloak.

### Database schema creation

Keycloak uses Liquibase to create the database schema and all constants (see [Creating the database](create_database.md)).  Liquibase supports PostgreSQL, but not directly CockroachDB. However, since CockroachDB uses the PostgreSQL jdbc driver and supports the syntax, Liquibase is mostly compatible with CockroachDB.

However, there are quite a few notable exceptions to this compatibility, and they arise when Keycloak's "normal" database changelogs are used. To correct the problem, a new set of changelogs specifically for CockroachDB are generated, mostly using the `ChangeLogEditor` - a java tool developped specifcally for this purpose.

The following subsections list the incompatibilities between the Keycloak changelogs and CockroachDB, and the modifications made.

#### Adding primary keys

In CockroachDB primary keys must be created with the tables and cannot be manipulated afterwards, but in the Keycloak changelogs the operation is always seperate. To correct this, the syntax is modified to group the creation of primary keys in the create table instruction.

However, in a few cases (about 8), "primary columns" are removed, and a new key is added. Since the table generation is in another file, in this case:

1. A new table is created with the correct structure
2. The contents are copied from the old table to the new table
3. The old table is dropped
4. The new table is renamed to the correct table name. 

#### Adding foreign keys

Foreign keys can be added and removed without problem in CockroachDB, however the column(s) MUST have a index defined on it. The solution creates indexes on all columns in a new changelog (to avoid precedence problems), and moves all "add foreign keys" to a third column (for the same reason.

#### Removing unique constraints

In CockroachDB altering a table to add a unique constraint in fact creates a unique index, and adds an entry to the list of constraints. And in fact creating a unique index does exactly the same thing, meaning that the two are interchangable. However, while it is possible to drop a unique index (which also drops the contraint), the CockRoachDB does not allow the syntax to drop the constraint. 

The modification changes all instances of "drop unique constraint" to "drop uinique index".

#### Others changes

We must only create indexes for foreign keys when one does not already exist. When a Foreign key is removed, the index must also be deleted.

#### Migration to Cockroach 2.X

After migration from Cockroach 1.8 to 2.0,the liquidbase script modifications were not compatible anymore. This may be due to a Cockroach regression.
The complexity of liquidbase scripts is due to the history of modification. All this history is not useful anymore and we created a liquidbase script with the needed schema for version 3.4.3 of Keycloak (All the addtion and remove of columns are no more presents).

### Transaction management

Contrary to standard SQL database as Postgres, CockroachDB does not perform locks on rows or tables to ensure consistency. Concurrent transactions may fails and it is the responsibilty of the software to issue the request again. To ensure comptabilty of Keycloak with CockorachDB, the main is to adapt the SessionServletFilter to issue again the SQL request if an exception occurs.

Note that Hibernate is not directly compatible with Cockroach, we use the savepoin-fix provided by CockroachDB to be able to use SAVEPOINT mechanism. However, we also have to hack hibernate to circumvent the rollbackOnly flag which is automatically set to true if an exection occurs during execution of SQL Native query. Our fix uses reflection to overrides rollbackOnly flag to true.
 

