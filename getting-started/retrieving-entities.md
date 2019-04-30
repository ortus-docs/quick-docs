# Retrieving Entities

Once you have an entity and its associated database table you can start retrieving data from your database.

## Active Record

You start every interaction with Quick with an instance of an entity. The easiest way to do this is using WireBox. `getInstance` is available in all handlers by default. WireBox can easily be injected in to any other class you need using `inject="wirebox"`.

Quick is backed by qb, a CFML Query Builder. With this in mind, think of retrieving records for your entities like interacting with qb. For example:

```javascript
var users = getInstance("User").all();

for (var user in users) {
    writeOutput(user.getUsername());
}
```

In addition to using `for` you can utilize the `each` function on arrays. For example:

```javascript
var users = getInstance("User").all();

prc.users.each(function(user) {
    writeOutput(user.getUsername());
});
```

You can add constraints to the query just the same as you would using qb directly:

```javascript
var users = getInstance("User")
    .where("active", 1)
    .orderBy("username", "desc")
    .limit(10)
    .get();
```

> For more information on what is possible with qb, check out the [qb documentation](https://qb.ortusbooks.com).

## Quick Service

A second way to retrieve results is to use a Quick Service. It is similar to a `VirtualEntityService` from cborm.

The easiest way to create a Quick Service is to inject it using the `quickService:` dsl:

```javascript
component {
    property name="userService" inject="quickService:User"
}
```

Any method you can call on an entity can be called on the service:

```javascript
var users = userService
    .where("active", 1)
    .orderBy("username", "desc")
    .limit(10)
    .get();
```

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

