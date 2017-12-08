# Creating the database structure in Keycloak

As long as an initial database is created, keycloak will be able to automatically create its initial state (tables, indexes, constants, etc...), without needing a administrator to run SQL scripts. It does this through the use of [Liquibase](http://www.liquibase.org/). The use of Liquibase also allows the database to be automatically modified when upgrading to a new version of Keycloak.

## About Liquibase

Full documentation may be found on [Liquibase's website](http://www.liquibase.org/documentation/index.html "Liquibase documentation"), but the basic principal is the following: changes to the database are recorded within _Database change logs_, XML files that contain the instructions for manipulating the database. One can for example create tables, delete tables, maniuplate column, indexes, constraints, etc... The philosophy is to create a set of one or more of such database change logs for every new release, and therefore no matter what version you are at, Liquibase can update from your current status to the new one.

In order to know which files have been run, and therefore which files to run in the event of an upgade, liquibase creates itself the `databasechangelog` table which records which changelogs have run, and the 

An important note is that liquibase uses the jdbc driver to determine which database is actually being used when applying the commands in the XML files to the database. When databases reuse drivers from another database, which is for example the case for [cockroachdb](cockroachdb.md). However, this means that the transformation Liquibase does from XML &rarr; SQL may be imperfect. 

## Use of Liquibase in Keycloak

In Keycloak all database changelog files can be found at `model/jpa/src/main/resources/META-INF`. The main file is `jpa-changelog-master.xml` which contains the list of all database change files to call, and the order in which they are called. Running Keycloak or its unit tests that access the DB will automatically run the master changelog, if necessary.

