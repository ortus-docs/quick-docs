# Query Scopes

## Definition

Query scopes are a way to encapsulate query constraints in your entities while
giving them readable names .

## A Practical Example

For instance, let's say that you need to write a report for subscribers to your
site. Maybe you track subscribers in a `users` table with a boolean flag in a
`subscribed` column. Additionally, you want to see the oldest subscribers first.
You keep track of when a user subscribed in a `subscribedDate` column. Your
query might look as follows:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .orderBy( "subscribedDate" )
    .get();
```

Now nothing is wrong with this query. It retrieves the data correctly and you
continue on with your day.

Later, you need to retrieve a list of subscribed users for a different part of
the site. So, you write a query like this:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .get();
```

We've duplicated the logic for how to retrieve active users now. If the database
representation changed, we'd have to change it in multiple places. For instance,
what if instead of keeping track of a boolean flag in the database, we just
checked that the `subscribedDate` column wasn't null?

```javascript
var subscribedUsers = getInstance( "User" )
    .whereNotNull( "subscribedDate" )
    .get();
```

Now we see the problem. Let's look at the solution.

The key here is that we are trying to retrieve subscribed users. Let's add a
scope to our `User` entity for `subscribed`:

```javascript
component extends="quick.models.BaseEntity" {

    function scopeSubscribed( query ) {
        return query.where( "subscribed", true );
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
        return query.where( "subscribed", true );
    }

}
```

Now our query is as follows:

```javascript
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .oldesetSubscribers()
    .get();
```

Best of all, we can reuse those scopes anywhere we see fit without duplicating
logic.

## Usage

All query scopes are methods on an entity that begin with the `scope` keyword.
You call these functions without the `scope` keyword \(as shown above\).

Each scope is passed the `query`, a reference to the current `QueryBuilder`
instance, as the first argument. Any other arguments passed to the scope will be
passed in order after that.

```javascript
component extends="quick.models.BaseEntity" {

    function scopeOfType( query, type ) {
        return query.where( "type", type );
    }

}
```

```javascript
var subscribedUsers = getInstance( "User" )
    .ofType( "admin" )
    .get();
```
