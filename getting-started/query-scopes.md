# Query Scopes

## Definition

Query scopes are a way to encapsulate query constraints in your entities while giving them readable names .

## A Practical Example

For instance, let's say that you need to write a report for subscribers to your site. Maybe you track subscribers in a `users` table with a boolean flag in a `subscribed` column. Additionally, you want to see the oldest subscribers first. You keep track of when a user subscribed in a `subscribedDate` column. Your query might look as follows:

```
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .orderBy( "subscribedDate" )
    .get();
```

Now nothing is wrong with this query. It retrieves the data correctly and you continue on with your day.

Later, you need to retrieve a list of subscribed users for a different part of the site. So, you write a query like this:

```
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .get();
```

We've duplicated the logic for how to retrieve active users now. If the database representation changed, we'd have to change it in multiple places. For instance, what if instead of keeping track of a boolean flag in the database, we just checked that the `subscribedDate` column wasn't null?

```
var subscribedUsers = getInstance( "User" )
    .whereNotNull( "subscribedDate" )
    .get();
```

Now we see the problem. Let's look at the solution.

The key here is that we are trying to retrieve subscribed users. Let's add a scope to our `User` entity for `subscribed`:

```
component extends="quick.models.BaseEntity" {

    function scopeSubscribed( query ) {
        return query.where( "subscribed", true );
    }

}
```

Now, we can use this scope in our query:

```
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .get();
```

We can use this on our first example as well, for our report.

```
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .orderBy( "subscribedDate" )
    .get();
```

We've successfully encapsulated our concept of a subscribed user!

We can add as many scopes as we'd like. Let's add one for `longestSubscribers`.

```
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

```
var subscribedUsers = getInstance( "User" )
    .subscribed()
    .longestSubscribers()
    .get();
```

Best of all, we can reuse those scopes anywhere we see fit without duplicating logic.

## Usage

All query scopes are methods on an entity that begin with the `scope` keyword. You call these functions without the `scope` keyword \(as shown above\).

Each scope is passed the `query`, a reference to the current `QueryBuilder` instance, as the first argument. Any other arguments passed to the scope will be passed in order after that.

```
component extends="quick.models.BaseEntity" {

    function scopeOfType( query, type ) {
        return query.where( "type", type );
    }

}
```

```
var subscribedUsers = getInstance( "User" )
    .ofType( "admin" )
    .get();
```

## Global Scopes

Occasionally, you want to apply a scope to each retrieval of an entity. An example of this is an Admin entity which is just a User entity with a type of admin. Global Scopes can be registered in the `applyGlobalScopes` method on an entity. Inside this entity you can call any number of scopes:

```
component extends="User" table="users" {

    function applyGlobalScopes() {
        this.ofType( "admin" );
    }

}
```

These scopes will be applied to the query without needing to call the scope again.

```
var admins = getInstance( "Admin" ).all();
// SELECT * FROM users WHERE type = 'admin'
```

If you have a global scope applied to an entity that you need to temporarily disable, you can disable them individually using the `withoutGlobalScope` method:

```
var admins = getInstance( "Admin" ).withoutGlobalScope( [ "ofType" ] ).all();
// SELECT * FROM users
```

## Subselects

Subselects are a useful way to grab data from related tables without having to execute the full relationship. Sometimes you just want a small piece of information like the `last_login_date` of a user, not the entire `Login` relationship. Subselects are perfect for this use case. You can even use subselects to provide the correct key for subselect relationships. We'll show how both work here.

Quick handles subselect properties \(or computed or formula properties\) through query scopes. This allows you to dynamically include a subselect. If you would like to always include a subselect, add it to your entity's [list of global scopes.](https://github.com/ortus-docs/quick-docs/tree/882521bd668671a1228bfffa5dbec7dcb78d0059/getting-started/getting-started/query-scopes.md#global-scopes)

Here's an example of grabbing the `last_login_date` for a User:

```
component extends="quick.models.BaseEntity" {

    /* properties */

    function logins() {
        return hasMany( "Login" );
    }

    function scopeWithLastLoginDate( query ) {
        addSubselect( "lastLoginDate", newEntity( "Login" )
            .select( "timestamp" )
            .whereColumn( "users.id", "user_id" )
            .latest()
        );
    }

}
```

We'd add this subselect by calling our scope:

```
var user = getInstance( "User" ).withLastLoginDate().first();
user.getLastLoginDate(); // {ts 2019-05-02 08:24:51}
```

We can even constrain our `User` entity based on the value of the subselect, so long as we've called the scope adding the subselect first \(or made it a global scope\).

```
 var user = getInstance( "User" )
     .withLastLoginDate()
     .where( "lastLoginDate", ">", "2019-05-10" )
     .all();
```

Or add a new scope to `User` based on the subselect:

```
function scopeLoggedInAfter( query, required date afterDate ) {
    return query.where( "lastLoginDate", ">", afterDate );
}
```

In this example, we are using the `addSubselect` helper method. Here is that function signature:

| Argument | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| name | string | `true` |  | The name for the subselect. This will be available as an attribute. |
| subselect | QueryBuilder OR Closure | `true` |  | Either a QueryBuilder object or a closure can be provided.  If a closure is provided it will be passed a query object as its only parameter.  The resulting query object will be used to computed the subselect. |

You might be wondering why not use the `logins` relationship? Or even `logins().latest().limit( 1 ).get()`? Because that executes a second query. Using a subselect we get all the information we need in one query, no matter how many entities we are pulling back.

Subselects can be used in conjunction with relationships to provide a dynamic, constrained relationship. In this example we will pull the latest post for a user.

```
component extends="BaseEntity" {

    /* properties */

    function scopeWithLatestPost( query ) {
        return addSubselect( "latestPostId", newEntity( "Post" )
            .select( "id" )
            .whereColumn( "user_id", "users.id" )
            .orderBy( "created_date", "desc" )
        ).with( "latestPost" );
    }

    function latestPost() {
        return belongsTo( "Post", "latestPostId" );
    }

}
```

This can be executed as follows:

```
var users = getInstance( "User" ).withLatestPost().all();
for ( var user in users ) {
    user.getLatestPost().getTitle(); // My awesome post, etc.
}
```

As you can see, we are loading the id of the latest post in a subquery and then using that value to eager load the `latestPost` relationship. This sequence will only execute two queries, no matter how many records are loaded.

