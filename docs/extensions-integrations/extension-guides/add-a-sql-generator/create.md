# Create a SqlGenerator

## Overview

When defining the execution logic for a particular `SqlGenerator`, the interface you are going to implement is [liquibase.sqlgenerator.SqlGenerator](https://javadocs.liquibase.com/liquibase-core/liquibase/sqlgenerator/SqlGenerator.html){:target="_blank"}.

!!! tip

    There is a [liquibase.sqlgenerator.core.AbstractSqlGenerator](https://javadocs.liquibase.com/liquibase-core/liquibase/sqlgenerator/core/AbstractSqlGenerator.html){:target="_blank"} base class you can use which limits the number of methods
    you must implement.

!!! tip

    When adding a SqlGenerator that provides database-specific logic, it's often best (but not required) to subclass the default SqlGenerator for that SqlStatement
    and override and change only the specific parts that need to be different while preserving the rest of the logic. 


!!! tip

    Specifying database-specific functionality is best done with `if (database instanceof ExampleDatabase)` blocks around the database-specific logic. 

    Using "instanceof" rather than "equals" allows the logic to apply to any subclasses (variants) of the given database.

## Implementing

### Empty Constructor

Like most Liquibase extensions, yourSqlGenerator must have an empty constructor.

### supports()

This method returns whether your SqlGenerator can be used for the given SqlStatement and Database. 

If you define your SqlGenerator class using generics, Liquibase will know the SqlStatement type you can support and not have to rely on supports() unless the SqlStatement class is correct. 
Therefore, you only have to check that the Database object is correct.

!!! warning

    Do not return `true` for SqlStatements your logic cannot handle. 

    The default implementation from `AbstractSqlGenerator` returns `true`, which is too broad for database-specific SqlGenerator implementations

### getPriority()

Is used as other [priority](../../extension-references/priority.md) values to control which SqlGenerator implementation to use out of all the ones that support the SqlStatement.

If unsure, use `PRIORITY_DEFAULT` for default/standard logic and `PRIORITY_DATABASE` for logic unique to a database. 

!!! tip

    When extending an existing SqlGenerator class, you **_must_** override getPriority() so it will be picked over the base implementation you are extending. 
    If you do not override this function they will both return the same priority and one will be chosen at random.

    To ensure a higher value is returned, `return super.getPriority() + 5`.

### generateSql()

Returns the SQL to run as [liquibase.sql.Sql](https://javadocs.liquibase.com/liquibase-core/liquibase/sql/Sql.html){:target="_blank"} objects.

The normal class to use for those `Sql` objects is `liquibase.sql.UnparsedSql`. 

This function returns an array of Sql statements because a single logical statement may require multiple SQL statements in some database. 
For example, `AddPrimaryKeyGenerator` may return a SQL call to add the primary key and another to reorganize the table. 

!!! tip

    **When extending an existing SqlGenerator**

    Depending on how different your SQL is from the base class you can either call `super.generateSql()` and modify the objects in the array before returning it, 
    OR create a new `Sql[]` array from scratch. 

    It tends to be best to modify the SQL generated by the parent class because that keeps you from having to duplicate all the
    argument handling logic that already exists in the parent. One approach is to call `super.generateSql()` and then create new `UnparsedSql` objects 
    using the `originalSql.toSql()` value passed through string replacements. 

    Also remember to always check your parent class's signature for methods it defines. It may have methods like `nullComesBeforeType()` which it uses to determine
    the final SQL and you simply have to override that method to get the correct SQL.

!!! tip

    When creating SQL, make sure you wrap object names in `database.escape...()` calls to correctly handle reserved words 
    and `database.escapeStringForDatabase()` around any string literals to handle special characters like `'` inside them.  

### validate()

Called to validate whether the SqlStatement is correctly configured. For example, if your logic requires the tableName to be set, add that to the `ValidationErrors` that is returned.

This method is always called, and so you do not need to re-validate configuration in other methods.

If you have no additional validation to perform vs. the base logic there is no need to override this method.

The `warn(SqlStatement, Database, SqlGeneratorChange)` method is similar to validate but returns non-fatal warnings to the user.

!!! tip

    When extending a base SqlGenerator, you can call `super.validate(statement, database, sqlGeneratorChain)` as a starting point, 
    then perform extra checks (or remove not-actually-invalid errors) before returning the [ValidationErrors](https://javadocs.liquibase.com/liquibase-core/liquibase/exception/ValidationErrors.html){:target="_blank"}.

### generateStatementsIsVolatile()

Ideally, your logic should be able to generate the correct SQL purely from the information stored in the SqlStatement object. 

However, if you cannot do that and instead need to check the database state or other external environments, you need to return `true` from this method to mark it as "volatile".
That lets Liquibase know that for commands such `updateSql`, it cannot rely on what your `generateSql` method returns because the internal database state is not actually correct.  


## Register your Class

Like all extensions, your SqlGenerator must be registered by adding your class name to `META-INF/services/liquibase.sqlgenerator.SqlGenerator`

## Example Code

```java
package com.example;

import liquibase.database.Database;
import liquibase.exception.ValidationErrors;
import liquibase.sql.Sql;
import liquibase.sql.UnparsedSql;
import liquibase.sqlgenerator.SqlGeneratorChain;
import liquibase.sqlgenerator.core.AbstractSqlGenerator;
import liquibase.statement.core.RenameColumnStatement;
import liquibase.structure.core.Column;

public class RenameColumnGeneratorExample extends AbstractSqlGenerator<RenameColumnStatement> {
    @Override
    public int getPriority() {
        return PRIORITY_DATABASE;
    }

    @Override
    public boolean supports(RenameColumnStatement statement, Database database) {
        return database instanceof ExampleDatabase;
    }

    @Override
    public ValidationErrors validate(RenameColumnStatement statement, Database database, SqlGeneratorChain sqlGeneratorChain) {
        ValidationErrors errors = new ValidationErrors();
        errors.checkRequiredField("tableName", statement.getTableName());
        errors.checkRequiredField("oldColumnName", statement.getOldColumnName());
        errors.checkRequiredField("newColumnName", statement.getNewColumnName());

        return errors;
    }

    @Override
    public Sql[] generateSql(RenameColumnStatement statement, Database database, SqlGeneratorChain sqlGeneratorChain) {
        return new Sql[]{
                new UnparsedSql("alter table " +
                        database.escapeTableName(statement.getCatalogName(), statement.getSchemaName(), statement.getTableName()) +
                        " rename column " +
                        database.escapeObjectName(statement.getOldColumnName(), Column.class) +
                        " to " +
                        database.escapeObjectName(statement.getNewColumnName(), Column.class))
        };
    }
}
```