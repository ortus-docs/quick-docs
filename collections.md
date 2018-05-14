# Collections

All queries that potentially return more than one record using Quick are returned using a `QuickCollection`. `QuickCollection` is a specialized version of [`CFCollection`](https://www.forgebox.io/view/cfcollection). It is a component that smooths over the various CFML engines to provide an extendible, reliable array wrapper with functional programming methods. You may be familiar with methods like `map` \(`ArrayMap`\), `filter` \(`ArrayFilter`\), or `reduce` \(`ArrayReduce`\). These methods work in every CFML engine with `CFCollection`.

Collections are more powerful than plain arrays. There are many methods that can make your work easier. For instance, let's say you needed to group each active user by the first letter of their username in a list.

```javascript
var users = getInstance( "User" ).all();

users
    .filter( function( user ) {
        return user.getActive();
    } )
    .pluck( "username" )
    .groupBy( function( username ) {
        return left( username, 1 );
    } );
```

So powerful! We think you'll love it.

## load

Additionally, `QuickCollection` includes a `load` method. `load` lets you eager load a relationship after executing the initial query.

```javascript
var posts = getInstance( "Post" ).all();

if ( someCondition ) {
    posts.load( "user" );
}
```

This is the same as if you had initially executed:

```javascript
getInstance( "Post" ).with( "user" ).all()
```

## $renderData

`QuickCollection` includes a `$renderData` method that lets you return a `QuickCollection` directly from your handler and translates the results and the entities within to a serialized version. Check out more about it in the [Serialization](serialization.md) chapter.

