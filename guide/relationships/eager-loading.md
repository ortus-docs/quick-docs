# Eager Loading

## The Problem

Let's imagine a scenario where you are displaying a list of posts. You fetch the posts:

```javascript
prc.posts = getInstance( "Post" ).limit( 25 ).get():
```

And start looping through them:

```markup
<cfoutput>
    <h1>Posts</h1>
    <ul>
        <cfloop array="#prc.posts#" item="post">
            <li>#post.getTitle()# by #post.getAuthor().getUsername()#</li>
        </cfloop>
    </ul>
</cfoutput>
```

When you visit the page, though, you notice it takes a while to load. You take a look at your SQL console and you've executed 26 queries for this one page! What?!?

Turns out that each time you loop through a post to display its author's username you are executing a SQL query to retreive that author. With 25 posts this becomes 25 SQL queries plus one initial query to get the posts. This is where the [N+1 problem](https://stackoverflow.com/questions/97197/what-is-n1-select-query-issue) gets its name.

So what is the solution? **Eager Loading.**

Eager Loading means to load all the needed users for the posts in one query rather than separate queries and then stitch the relationships together. With Quick you can do this with one method call.

## The Solution

### with

You can eager load a relationship with the `with` method call.

```javascript
prc.posts = getInstance( "Post" )
    .with( "author" )
    .limit( 25 )
    .get();
```

`with` takes one parameter, the name of the relationship to load. Note that this is the name of the function, not the entity name. For example:

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function author() {
        return belongsTo( "User" );
    }

}
```

To eager load the User in the snippet above you would call pass `author` to the `with` method.

```javascript
getInstance( "Post" ).with( "author" ).get();
```

For this operation, only two queries will be executed:

```
SELECT * FROM `posts` LIMIT 25

SELECT * FROM `users` WHERE `id` IN (1, 2, 3, 4, 5, 6, ...)
```

Quick will then stitch these relationships together so when you call `post.getAuthor()` it will use the fetched relationship value instead of going to the database.

### Nested Relationships

You can eager load nested relationships using dot notation. Each segment must be a valid relationship name.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function country() {
        return belongsTo( "User" );
    }

}
```

```javascript
getInstance( "Post" ).with( "author.country" );
```

You can eager load multiple relationships by passing an array of relation names to `with` or by calling `with` multiple times.

```javascript
getInstance( "Post" ).with( [ "author.country", "tags" ] );
```

### Constraining Eager Loaded Relationships

In most cases when you want to constrain an eager loaded relationship, the better approach is to create a new relationship.

```javascript
// User.cfc
component {

    function posts() {
        return hasMany( "Post" );
    }

    function publishedPosts() {
        return hasMany( "Post" ).published(); // published is a query scope on Post
    }

}
```

You can eager load either option.

```javascript
getInstance( "User" ).with( "posts" ).get();
getInstance( "User" ).with( "publishedPosts" ).get();
```

Occassionally that decision needs to be dynamic. For example, maybe you only want to eager load the posts created within a timeframe defined by a user. To do this, pass a struct instead of a string to the `with` function. The key should be the name of the relationship and the value should be a function. This function will accept the related entity as its only argument. Here is an example:

```javascript
getInstance( "User" ).with( { "posts" = function( query ) {

} } ).latest().get();
```

If you need to load nested relationships with constraints you can call `with` in your constraint callback to continue eager loading relationships.

```javascript
getInstance( "User" ).with( { "posts" = function( q1 ) {
    return q1
        .whereBetween( "published_date", rc.startDate, rc.endDate )
        .with( { "comments" = function( q2 ) {
            return q2.where( "body", "like", rc.search );
        } } );
} } ).latest().get();
```

### load

Finally, you can postpone eager loading until needed by using the `load` method on `QuickCollection`. `load` has the same function signature as `with`. `QuickCollection` is the object returned for all Quick queries that return more than one record. Read more about it in [Collections](../collections.md).

### Preventing Lazy Loading

The N+1 problem can be hard to catch.  One missed eager loading here and another there and soon your app feels slow and unresponsive.  There are good tools for catching these things, like Quick's [log output](../debugging.md#logbox-appender) and [cbDebugger](../debugging.md#cbdebugger).  Another tool is preventing lazy loading entirely.

First, you can set this globally for your application using module settings.

```cfscript
component {

    function configure() {
        moduleSettings = {
            "quick": {
                "preventLazyLoading": false
            }
        };
    }


    function staging() {
        moduleSettings.quick.preventLazyLoading = true;
    }
    
    function development() {
        moduleSettings.quick.preventLazyLoading = true;
    }
    
    function testing() {
        moduleSettings.quick.preventLazyLoading = true;
    }

}
```

Utilizing [ColdBox's environment detection](https://coldbox.ortusbooks.com/getting-started/configuration/coldbox.cfc/configuration-directives/environments), we can turn this on only in non-production environments. This gives us the benefit of failing loudly while we are developing while being safe to turn on in existing applications.

Additionally, this can be turned on for a single query using the `preventLazyLoading`function. (If you really want to, you can also use `allowLazyLoading`to override this protection.)

By default, trying to lazy load a relationship with this feature enabled will through a `QuickLazyLoadingException`.  You can customize what happens when the no-lazy loading policy is violated by setting the `lazyLoadingViolationCallback` setting.

```cfscript
component {

    function configure() {
        moduleSettings = {
            "quick": {
                "lazyLoadingViolationCallback": ( entity, relationName ) => {
                    log.warn( "Lazy Loading detected: [#entity.mappingName()#] accessing [#relationName#]" );
                }
            }
        };
    }
    
}
```

You can also set this callback per-entity by passing in the callback to the `preventLazyLoading` function.

### Preconfigured (Default) Eager Loading

Entities can declare relationships that should be eager loaded on every query by assigning an array of relationship paths to `variables._with`:

```cfscript
component name="Post" extends="quick.models.BaseEntity" accessors="true" {

	variables._with = [ "author", "comments.author" ];

	function author() {
		return belongsTo( "User" );
	}

	function comments() {
		return hasMany( "Comment" );
	}

}
```

The paths use the same dot notation as the query builder's `with()` method, so nested relationships can be preconfigured. Quick applies these relationships whenever it creates a new query for the entity, including calls such as `all()`, `get()`, `first()`, and `find()`.

Preconfigured eager loading is most useful for relationships that nearly every consumer needs. Every configured relationship adds work to each entity query and can retrieve substantially more data than the caller needs. For relationships used only by specific operations, prefer an explicit query-level call:

```cfscript
getInstance( "Post" ).with( "comments" ).get();
```

{% hint style="warning" %}
Use preconfigured eager loading sparingly. It can add queries and over-fetch data for consumers that do not need the configured relationships.
{% endhint %}
