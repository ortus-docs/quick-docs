# Retrieving Relationships

Relationships can be used in two ways.

The first is as a getter. Calling `user.getPosts()` will execute the relationship, cache the result, and return it.

```javascript
var posts = user.getPosts();
```

The second is as a relationship. Calling `user.posts()` returns a `Relationship` instance to retrieve the posts that can be further constrained. A `Relationship` is backed by qb as well, so feel free to call any qb method to further constrain the relationship.

```javascript
var newestPosts = user
    .posts()
    .orderBy( "publishedDate", "desc" )
    .get();
```

You can also call the other Quick fetch methods: `first`, `firstOrFail`, `find`, `findOrFail`, and `firstWhere` are all supported.  This is especially useful to constrain the entities available to a user by using the user's relationships:

```javascript
// This will only find posts the user has written.
var post = user.posts().findOrFail( rc.id );
```

## Assigning a Loaded Relationship

Use `assignRelationship` when you already have a related value and want the relationship accessor to return it without executing a relationship query. This is especially useful after creating related records:

```javascript
var user = getInstance( "User" ).findOrFail( 1 );
var post = user.posts().create( { "body" : "A new post" } );

// getPosts() now returns this array without querying the database.
user.assignRelationship( "posts", [ post ] );
```

Pass a Quick entity for a singular relationship and an array for a collection relationship. Assigning a value replaces any previously loaded value and marks the relationship as loaded.

`assignRelationship` only changes the in-memory entity. It does not save either entity, update foreign keys, attach pivot records, or validate that the value matches the relationship type.

Call `clearRelationship( "posts" )` to discard the assigned value and its loaded marker. The next relationship accessor call can then lazy load the relationship normally, when lazy loading is enabled.

You can also use other Quick fetch methods that provide new entities if a related entity is not found, such as `firstOrNew`, `firstOrCreate`, `findOrNew`, and `findOrCreate`.

```javascript
var post = user.posts().firstOrNew( { "title": "First Post" } );
```

You can also get a new unloaded related entity by calling either the `newEntity` or the `fill` functions.

```javascript
var newPost = user.posts().fill( { "title": "My new post" } );
```

### loadRelationship

The code above is a shortcut for calling `loadRelationship` and `get{RelationName}` on an entity.

```
public any function loadRelationship( required any name, boolean force = false, boolean parallel = false )
```
