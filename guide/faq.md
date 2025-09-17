# FAQ

## What's the difference between \`posts()\` and \`getPosts()\`?

_Answered by_ [_Eric Peterson_](https://github.com/elpete)

{% hint style="info" %}
**TLDR:** Calling a relationship method returns a relationship component. Preceding that call with `get` loads and executes the relationship query.
{% endhint %}

When you define a relationship you name the function without a `get` in front of it. When calling a relationship with `get` preceding it, Quick loads the relationship and executes the query. You are returned either a single entity (or `null`) or an array of entities.

When you call the relationship function you get back an instance of a Quick Relationship component. This component is configured based on the entity that created it and the attributes configured in the relationship call. You can think of a relationship component as a super-charged query. In fact, you can call other Quick and qb methods on the relationship object. This is one way to restrict the results you get back.

For instance, perhaps you want to retrieve a specific Post by its id. In this case, you want the Post to be found only if it belongs to the User. You could add a constraint to a Post query on the foreign key `userId` like so:

```javascript
getInstance( "Post" )
    .where( "userId", prc.loggedInUser.getId() )
    .findOrFail( rc.id );
```

Another way to write this is by leveraging existing relationships:

```javascript
prc.loggedInUser.posts().findOrFail( rc.id );
```

Let's disect this. At first glance it may look like it is just a matter of style and preference. But the power behind the relationship approach is that it encapsulates the logic to define the relationship. Remember that relationships don't have to only define foreign keys. They can define any custom query logic and give a name to it. They can even build on each other:

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasMany( "Post" );
    }
    
    function publishedPosts() {
        return this.posts().whereNotNull( "publishedDate" );
    }
    
}
```

You see here that we have now named an important concept to our domain - a published post. Let's take this one step further and name the query logic on the Post entity itself.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function scopePublished( qb ) {
        qb.whereNotNull( "publishedDate" );   
    }

}
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasMany( "Post" );
    }
    
    function publishedPosts() {
        return this.posts().published();
    }
    
}
```

## Why do interception points on subclassed entities fire twice?

{% hint style="info" %}
**TLDR:** Quick will call service methods first on the parent entity and then on the child entity. Both instances will fire inception point events.
{% endhint %}

When working with subclassed entities and you call a method that would change that state of the database ( such as `.save()` or `.delete()` ) quick will first retrieve an instance of the parent entity and perform the same method call on that instance before calling the method on the child class. Interception points will subsequently be fired for both method calls. To overcome this, you can explicitly check which entity was used to fire events in the code. Quick will pass the `entityName` in the eventData argument of the interception point which can be used to check which instance fired the event.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {
    
    function preSave( event, eventData, buffer, rc, prc ) {
        if ( arguments.eventData.entityName == "User" ) {
            // this code will only execute once when interacting with Admin.cfc
        }
    }
    
}

// Admin.cfc
component extends="models.User" {

}
```

## Can I access qb to run an SQL statement?

Quick is powered by qb [qb documentation.](https://qb.ortusbooks.com/) and can be accessed from within Quick using either the `getQB` or `retrieveQuery` methods. For your convenience, the qb builder will have already populated the database table and column names for you in the returned qb instance.

```cfml
qb = getInstance( "User" ).getQB();
// OR
qb = getInstance( "User" ).retrieveQuery();
```

## What's the difference between retrieveAttribute( "x" ) and getX()?

`getX` uses onMissingMethod to call `retrieveAttribute`. This is so you can create your own custom getters.

## When do I use a scope method and when do I use a normal method?

## I keep getting a \`QuickEntityNotLoaded\` exception. What is the difference between a loaded entity and an unloaded entity?

## How can I add a subselect field to my entity?

## How can I add a computed field to my entity, like from a SQL CASE statement?

## How can I always add a subselect or computed field to my queries?
