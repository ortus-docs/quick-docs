# Query Scopes and Subselects

## What are Scopes?

Query scopes are a way to encapsulate query constraints in your entities while giving them readable names .

### A Practical Example

For instance, let's say that you need to write a report for subscribers to your site. Maybe you track subscribers in a `users` table with a boolean flag in a `subscribed` column. Additionally, you want to see the oldest subscribers first. You keep track of when a user subscribed in a `subscribedDate` column. Your query might look as follows:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .orderBy( "subscribedDate" )
    .get();
```

Now nothing is wrong with this query. It retrieves the data correctly and you continue on with your day.

Later, you need to retrieve a list of subscribed users for a different part of the site. So, you write a query like this:

```javascript
var subscribedUsers = getInstance( "User" )
    .where( "subscribed", true )
    .get();
```

We've duplicated the logic for how to retrieve active users now. If the database representation changed, we'd have to change it in multiple places. For instance, what if instead of keeping track of a boolean flag in the database, we just checked that the `subscribedDate` column wasn't null?

```javascript
var subscribedUsers = getInstance( "User" )
    .whereNotNull( "subscribedDate" )
    .get();
```

Now we see the problem. Let's look at the solution.

The key here is that we are trying to retrieve subscribed users. Let's add a scope to our `User` entity for `subscribed`:

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

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
component extends="quick.models.BaseEntity" accessors="true" {

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
    .longestSubscribers()
    .get();
```

Best of all, we can reuse those scopes anywhere we see fit without duplicating logic.

### Usage

All query scopes are methods on an entity that begin with the `scope` keyword. You call these functions without the `scope` keyword \(as shown above\).

Each scope is passed the `query`, a reference to the current `QuickBuilder` instance, as the first argument. Any other arguments passed to the scope will be passed in order after that.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

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

### Scopes that Return Values

All of the examples so far either returned the `QuickBuilder` object or nothing. Doing so lets you continue to chain methods on your Quick entity. If you instead return a value, Quick will pass on that value to your code. This lets you use scopes as shortcut methods that work on a query.

For example, maybe you have a domain method to reset passwords for a group of users, and you want the count of users updated returned.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="username";
    property name="password";
    property name="type";

    function scopeOfType( query, type = "limited" ) {
        return query.where( "type", type );
    }

    function scopeResetPasswords( query ) {
        return query.updateAll( { "password" = "" } ).result.recordcount;
    }

}
```

```javascript
getInstance( "User" ).ofType( "admin" ).resetPasswords(); // 1
```

## Global Scopes

Occasionally, you want to apply a scope to each retrieval of an entity. An example of this is an Admin entity which is just a User entity with a type of admin. Global Scopes can be registered in the `applyGlobalScopes` method on an entity. Inside this entity you can call any number of scopes:

```javascript
component extends="User" table="users" accessors="true" {

    function applyGlobalScopes() {
        this.ofType( "admin" );
    }

}
```

These scopes will be applied to the query without needing to call the scope again.

```javascript
var admins = getInstance( "Admin" ).all();
// SELECT * FROM users WHERE type = 'admin'
```

If you have a global scope applied to an entity that you need to temporarily disable, you can disable them individually using the `withoutGlobalScope` method:

```javascript
var admins = getInstance( "Admin" ).withoutGlobalScope( [ "ofType" ] ).all();
// SELECT * FROM users
```

## Subselects

Subselects are a useful way to grab data from related tables without having to execute the full relationship. Sometimes you just want a small piece of information like the `lastLoginDate` of a user, not the entire `Login` relationship. Subselects are perfect for this use case. You can even use subselects to provide the correct key for dynamic subselect relationships. We'll show how both work here.

Quick handles subselect properties \(or computed or formula properties\) through query scopes. This allows you to dynamically include a subselect. If you would like to always include a subselect, add it to your entity's [list of global scopes.](https://github.com/ortus-docs/quick-docs/blob/2.1.0/getting-started/query-scopes.md#global-scopes)

Here's an example of grabbing the `lastLoginDate` for a User:

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    /* properties */

    function logins() {
        return hasMany( "Login" ).latest();
    }

    function scopeAddLastLoginDate( query ) {
        addSubselect( "lastLoginDate", newEntity( "Login" )
            .select( "timestamp" )
            .whereColumn( "users.id", "user_id" )
        );
    }

}
```

We'd add this subselect by calling our scope:

```javascript
var user = getInstance( "User" ).addLastLoginDate().first();
user.getLastLoginDate(); // {ts 2019-05-02 08:24:51}
```

We can even constrain our `User` entity based on the value of the subselect, so long as we've called the scope adding the subselect first \(or made it a global scope\).

```javascript
 var user = getInstance( "User" )
     .addLastLoginDate()
     .where( "lastLoginDate", ">", "2019-05-10" )
     .all();
```

Or add a new scope to `User` based on the subselect:

```javascript
function scopeLoggedInAfter( query, required date afterDate ) {
    return query.where( "lastLoginDate", ">", afterDate );
}
```

In this example, we are using the `addSubselect` helper method. Here is that function signature:

| Argument | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| name | string | `true` |  | The name for the subselect. This will be available as an attribute. |
| subselect | string OR QueryBuilder OR Function | `true` |  | Either a dot-delimited string representing a relationship chain ending with an attribute name, a QueryBuilder object, or a closure that configures a QueryBuilder object can be provided.  If a closure is provided it will be passed a query object as its only parameter.  The resulting query object will be used to computed the subselect. |

You might be wondering why not use the `logins` relationship? Or even `logins().latest().limit( 1 ).get()`? Because that executes a second query. Using a subselect we get all the information we need in one query, no matter how many entities we are pulling back.

### Using Relationships in Subselects

In most cases the values you want as subselects are values from your entity's relationships. In these cases, you can use a shortcut to define your subselect in terms of your entity's relationships represented as a dot-delimited string.

Let's re-write the above subselect for `lastLoginDate` for a User using the existing relationship:

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    /* properties */

    function logins() {
        return hasMany( "Login" );
    }

    function scopeAddLastLoginDate( query ) {
        addSubselect( "lastLoginDate", "logins.timestamp" );
    }

}
```

Much simpler! In addition to be much simpler this code is also more dynamic and reusable. We have a relationship defined for logins if we need to fetch them. If we change how the `logins` relationship is structured, we only have one place we need to change.

With the query cleaned up using existing relationships, you might find yourself adding subselects directly in your handlers instead of behind scopes. This is fine in most cases. Keep an eye on how many places you use the subselect in case you need to re-evaluate and move it behind a scope.

```javascript
var user = getInstance( "User" )
    .addSubselect( "lastLoginDate", "logins.timestamp" )
    .first();
user.getLastLoginDate(); // {ts 2019-05-02 08:24:51}
```
{% hint style="success" %}
`addSubSelect` call must come after a `select` clause. This is because the `select` method overwrites the selected columns. If you use `selectRaw` or `addSelect`, this will not affect you. See below example.
{% endhint %}

```javascript
var user = getInstance( "User" )
    // Select clause specified
    .select( "firstName" )
    // addSubSelect after select clause
    .addSubselect( "lastLoginDate", "logins.timestamp" )
    .first();
user.getLastLoginDate(); // {ts 2019-05-02 08:24:51}
```

### Dynamic Subselect Relationships

Subselects can be used in conjunction with relationships to provide a dynamic, constrained relationship. In this example we will pull the latest post for a user.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

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

```javascript
var users = getInstance( "User" ).withLatestPost().all();
for ( var user in users ) {
    user.getLatestPost().getTitle(); // My awesome post, etc.
}
```

As you can see, we are loading the id of the latest post in a subquery and then using that value to eager load the `latestPost` relationship. This sequence will only execute two queries, no matter how many records are loaded.

## Virtual Attributes

Virtual attributes are attributes that are not present on the table backing the Quick entity. A Subselect is an example of a virtual attribute. Other examples could include calculated counts or `CASE` statement results.

By default, if you add a virtual column to a Quick query, you won't see anything in the entity. This is because Quick needs to have an attribute defined to map the result to. You can create a virtual attribute in these cases.

{% hint style="warning" %}
This step is unnecessary when using the `addSubselect` helper method.
{% endhint %}

Here's an example including the result of a `CASE` statement as a field:

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function scopeAddType( qb ) {
        qb.selectRaw( "
                CASE
                    WHEN publishedDate IS NULL THEN 'unpublished'
                    ELSE 'published'
                END AS publishedStatus
            " );
        appendVirtualAttribute( "publishedStatus" );
    }

}
```

With this code, we could now access the `publishedStatus` just like any other attribute. It will not be updated, inserted, or saved though, as it is just a virtual column.

The `appendVirtualAttribute` method adds the given name as an attribute available in the entity.

### appendVirtualAttribute

| Argument | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| name | string | `true` |  | The attribute name to create. |

Creates a virtual attribute for the given name.

{% hint style="success" %}
It is likely that Quick will introduce more helper methods in the future making these calls simpler.
{% endhint %}

