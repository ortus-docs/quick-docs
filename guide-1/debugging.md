# Debugging

## Debugging a Single Query

### toSQL

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| showBindings | boolean | `false` | ​`false` | If true, the bindings for the query will be substituted back in where the question marks \(`?`\) appear. |

Returns the SQL that would be executed for the current query.

```javascript
var userQuery = getInstance( "User" )
    .where( "active", "=", 1 );

writeOutput( userQuery.toSQL() );
```

```sql
SELECT `users`.`id`, `users`.`username` /* etc */
FROM "users"
WHERE "active" = ?
```

The bindings for the query are represented by question marks \(`?`\) just as when using `queryExecute`.  qb can replace each question mark with the corresponding `cfqueryparam`-compatible struct by passing `showBindings = true` to the method.

```javascript
var userQuery = getInstance( "User" )
    .where( "active", "=", 1 );

writeOutput( userQuery.toSQL( showBindings = true ) );
```

{% code title="Result" %}
```sql
SELECT `users`.`id`, `users`.`username` /* etc */
FROM "users"
WHERE "active" = {"value":1,"cfsqltype":"CF_SQL_NUMERIC","null":false}
```
{% endcode %}

### tap

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| callback | Function | `true` | ​ | A function to execute with an instance of the current entity. |

Executes a callback with the current entity passed to it.  The return value from `tap` is ignored and the current entity is returned.

While not strictly a debugging method, `tap` makes it easy to see the changes to an entity after each call without introducing temporary variables.

```javascript
getInstance( "User" )
    .tap( function( e ) {
        writeOutput( e.toSQL() & "<br>" );
    } )
    .where( "active", "=", 1 )
    .tap( function( e ) {
        writeOutput( e.toSQL() & "<br>" );
    } );
```

```sql
SELECT `users`.`id`, `users`.`username` /* etc */ FROM "users"
SELECT `users`.`id`, `users`.`username` /* etc */ FROM "users" WHERE "active" = ?
```

## Debugging All Queries

### cbDebugger

Starting in [cbDebugger](https://forgebox.io/view/cbdebugger) 2.0.0 you can view all your Quick and qb  queries for a request.  This is the same output as using qb standalone.  This is enabled by default if you have qb installed.  Make sure your debug output is configured correctly and scroll to the bottom of the page to find the debug output.

Additionally, with Quick installed you will see number of loaded entities for the request.  This can help identify places that are missing pagination or relationships that could be tuned or converted to a subselect.

![](../.gitbook/assets/2020-05-04_12-40.png)

### LogBox Appender

Quick is set to log all queries to a debug log out of the box via qb.  To enable this behavior, configure LogBox to allow debug logging from qb's grammar classes.

{% code title="config/ColdBox.cfc" %}
```sql
logbox = {
    debug = [ "qb.models.Grammars" ]
};
```
{% endcode %}

{% hint style="info" %}
qb can be quite chatty when executing many database queries.  Make sure that this logging is only enabled for your development environments using [ColdBox's environment controls](https://coldbox.ortusbooks.com/getting-started/configuration/coldbox.cfc/configuration-directives/environments).
{% endhint %}

### Interception Points

ColdBox Interception Points can also be used for logging, though you may find it easier to use LogBox.  See the documentation for [qb's Interception Points](https://qb.ortusbooks.com/query-builder/options-and-utilities/interception-points)  or Quick's [own interception points](interception-points.md) for more information.

