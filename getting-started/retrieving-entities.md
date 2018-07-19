# Retrieving Entities

Once you have an entity and its associated database table you can start retrieving data from your database.

You start every interaction with Quick with an instance of an entity. The easiest way to do this is using WireBox. `getInstance` is available in all handlers by default. WireBox can easily be injected in to any other class you need using `inject="wirebox"`.

Quick is backed by qb, a CFML Query Builder. With this in mind, think of retrieving records for your entities like interacting with qb. For example:

```javascript
var users = getInstance( "User" ).all();

// users is a QuickCollection.  To get an array,
// first call `get` or `toArray`
for ( var user in users.get() ) {
    writeOutput( user.getUsername() );
}
```

In addition to using `for` you can utilize the `each` function on `QuickCollection`. For example:

```javascript
var users = getInstance( "User" ).all();

prc.users.each( function( user ) {
    writeOutput( user.getUsername() );
} );
```

You can add constraints to query just the same as you would using qb directly:

```javascript
var users = getInstance( "User" )
    .where( "active", 1 )
    .orderBy( "username", "desc" )
    .limit( 10 )
    .get();
```

> For more information on what is possible with qb, check out the [qb documentation](https://qb.ortusbooks.com).

## Collections

Queries that return more than one record return a `QuickCollection`, a type of `CFCollection`. Read more about [collections here](../collections.md).

## Aggregates

Calling qb's aggregate methods \(`count`, `max`, etc.\) will return the appropriate value instead of an entity or collection of entities.

## Custom Quick Retrieval Methods

There are a few custom retrieval methods for Quick:

### all

Retrieves all the records for an entity. Calling `all` will ignore any constraints on the query.

### findOrFail & firstOrFail

These two methods will throw a `EntityNotFound` exception if the query returns no results.

The `findOrFail` method should be used in place of `find`, passing an id in to retrieve.

The `firstOrFail` method should be used in place of `first`, being called after constraining a query.

