# Defining An Entity

To get started with Quick, you need an entity. You start by extending
`quick.models.BaseEntity`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {}
```

That's all that is needed to get started with Quick. There are a few defaults of
Quick worth mentioning here.

## Tables

We don't need to tell Quick what table name to use for our entity. By default,
Quick uses the pluralized name of the component for the table name. That means
for our `User` entity Quick will assume the table name is `users`. You can
override this by specifying a `table` metadata attribute on the component.

```javascript
// User.cfc
component table="my_users" extends="quick.models.BaseEntity" {}
```

## Primary Key

By default, Quick assumes a primary key of `id`. The name of this key can be
configured by setting `variables._key` in your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    variables._key = "user_id";

}
```

Quick also assumes a key type that is auto-incrementing. If you would like a
different key type, define a function called \`keyType\` and return the key type
from that function.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function keyType() {
        return variables._wirebox.getInstance( "UUIDKeyType@quick" );
    }

}
```

Quick ships with the following key types:

+ `AutoIncrementingKeyType`
+ `NullKeyType`
+ `ReturningKeyType`
+ `UUIDKeyType`

`keyType` can be any component that adheres to the `keyType` interface, so feel
free to create your own and distribute them via ForgeBox.

```javascript
interface displayname="KeyType" {

    /**
    * Called to handle any tasks before inserting into the database.
    * Recieves the entity as the only argument.
    */
    public void function preInsert( required entity );

    /**
    * Called to handle any tasks after inserting into the database.
    * Recieves the entity and the queryExecute result as arguments.
    */
    public void function postInsert( required entity, required struct result );

}
```

## Columns

You specify what columns are retrieved by adding properties to your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    property name="id";
    property name="username";
    property name="email";

}
```

Now, only the `id`, `username`, and `email` columns will be retrieved.

> Note: Make sure to include the primary key \(`id` by default\) as a property.

To prevent Quick from mapping a property to the database add the
`persistent="false"` attribute to the property.

```text
// User.cfc
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    property name="bcrypt" inject="@BCrypt" persistent="false";

    property name="id";
    property name="username";
    property name="email";

}
```

If the column name in your table is not the column name you wish to use in
quick, you can alias it using the `column` metadata attribute.

```javascript
component extends="quick.models.BaseEntity" {

    property name="id";
    property name="username" column="user_name";
    property name="countryId" column="FK_country_id";

}
```

## Multiple datasource support

Quick uses a default datasource and default grammar, as described
[here](./README.md). If you are using multiple datasources you can override
default datasource by specifying a `datasource` metadata attribute on the
component. If your extra datasource has a different grammar you can override
your grammar as well by specifying a `grammar` attribute.

```javascript
// User.cfc
component datasource="myOtherDatasource" grammar="PostgresGrammar" extends="quick.models.BaseEntity" {}
```

At the time of writing Valid grammar options are: `MySQLGrammar`,
`PostgresGrammar`, `MSSQLGrammar` and `OracleGrammar`. Please check the
[qb docs](https://qb.ortusbooks.com/) for additional options.
