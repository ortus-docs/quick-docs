# FAQ

## What's the difference between \`posts()\` and \`getPosts()\`?

_Answered by_ [_Eric Peterson_](https://github.com/elpete)__

{% hint style="info" %}
**TLDR:** Calling a relationship method returns a relationship component.  Preceding that call with `get` loads and executes the relationship query.
{% endhint %}

When you define a relationship you name the function without a `get` in front of it.  When calling a relationship with `get` preceding it, Quick loads the relationship and executes the query.  You are returned either a single entity (or `null`) or an array of entities.

When you call the relationship function you get back an instance of a Quick Relationship component.  This component is configured based on the entity that created it and the attributes configured in the relationship call.  You can think of a relationship component as a super-charged query.  In fact, you can call other Quick and qb methods on the relationship object.  This is one way to restrict the results you get back.

For instance, perhaps you want to retrieve a specific Post by its id.  In this case, you want the Post to be found only if it belongs to the User.  You could add a constraint to a Post query on the foreign key `userId` like so:

```javascript
getInstance( "Post" )
    .where( "userId", prc.loggedInUser.getId() )
    .findOrFail( rc.id );
```

Another way to write this is by leveraging existing relationships:

```javascript
prc.loggedInUser.posts().findOrFail( rc.id );
```

Let's disect this.  At first glance it may look like it is just a matter of style and preference. But the power behind the relationship approach is that it encapsulates the logic to define the relationship.  Remember that relationships don't have to only define foreign keys.  They can define any custom query logic and give a name to it.  They can even build on each other:

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

You see here that we have now named an important concept to our domain - a published post.  Let's take this one step further and name the query logic on the Post entity itself.

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

## When do I use a scope method and when do I use a normal method?

## I keep getting a \`QuickEntityNotLoaded\` exception.  What is the difference between a loaded entity and an unloaded entity?

## How can I add a subselect field to my entity?

## How can I add a computed field to my entity, like from a SQL CASE statement?

## How can I always add a subselect or computed field to my queries?
