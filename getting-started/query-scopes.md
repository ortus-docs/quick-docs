# Query Scopes

## Definition

Query scopes are a way to encapsulate query constraints in your entities while giving them readable names .

## A Practical Example

For instance, let's say that you need to write a report for subscribers to your site. Maybe you track subscribers in a `users` table with a boolean flag in a `subscribed` column. Additionally, you want to see the oldest subscribers first. You keep track of when a user subscribed in a `subscribedDate` column. Your query might look as follows:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", 1 )
    .orderBy( "subscribedDate" )
    .get();
```

Now nothing is wrong with this query. It retrieves the data correctly and you continue on with your day.

Later, you need to retrieve a list of subscribed users for a different part of the site. So, you write a query like this:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", 1 )
    .get();
```

We've duplicated the logic for how to retrieve active users now. If the database representation changed, we'd have to change it in multiple places. For instance, what if instead of keeping track of a boolean flag in the database, we just checked that the `subscribedDate` column wasn't null?

```javascript
var subscribedUsers = getInstance( "User" )
    .whereNotNull( "subscribedDate" )
    .get()
```

Now we see the problem. Let's look at the solution.

The key here is that we are trying to retrieve subscribed users. Let's add a scope to our `User` entity for `subscribed`:

```javascript
component extends="quick.models.BaseEntity" {

    function scopeSubscribed( query ) {
        return query.where( "subscribed", 1 );
    }

}
```

Now, we can use this scope in our query:

```javascript
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .get();
```

We can use this on our first example as well, for our report.

```javascript
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .orderBy( "subscribedDate" )
    .get();
```

We've successfully encapsulated our concept of a subscribed user!

We can add as many scopes as we'd like. Let's add one for `longestSubscribers`.

```javascript
component extends="quick.models.BaseEntity" {

    function scopeLongestSubscribers( query ) {
        return query.orderBy( "subscribedDate" );
    }

    function scopeSubscribed( query ) {
        return query.where( "subscribed", 1 );
    }

}
```

Now our query is as follows:

```javascript
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .longestSubscribers()
    .get();
```

Best of all, we can reuse those scopes anywhere we see fit without duplicating logic. We can even use one scope in defining a second scope. We could make `longestSubscribers` presume `subscribed()` rather than having to explicitly call both scopes:

```javascript
component extends="quick.models.BaseEntity" {

    function scopeSubscribed( query ) {
        return query.where( "subscribed", 1 );
    }
    
    function scopeLongestSubscribers( query ) {
        return this.scopeSubscribed( query ).orderBy( "subscribedDate" );
    }
    
}
```
Then our queries become one line shorter:

```javascript
var subscribedUsers = getInstance( "User" )
    .longestSubscribers()
    .get();
```

Note that, when making a "sub-scope" on a Quick entity like the above example, you must reference the complete name of the parent scope function (e.g. `scopeSubscribed`) and pass in the query argument. This is an exception to ordinary `Quick` practice of using just the name of the scope, which you can continue to do when calling a scope from any location other than the entity itself.

## Usage

All query scopes are methods on an entity that begin with the `scope` keyword. You call these functions without the `scope` keyword \(as shown above\).

Each scope is passed two arguments: `query`, a reference to the current `QueryBuilder` instance; and `args`, any arguments passed to the scope call.

