# Retrieving Relationships

Relationships can be used in two ways.

The first is as a getter. Calling `user.getPosts()` will execute the relationship, cache the result, and return it.

```javascript
var posts = user.getPosts();
```

For new, unloaded entities, relationship getters do not execute a query. To-one relationships return `null` or their configured default entity, while collection relationships return an empty collection.

```javascript
var user = getInstance( "User" );

user.getProfile(); // null
user.getPosts(); // []
```

You can seed an unloaded relationship directly with `retrieveRelationship()`:

```javascript
user.retrieveRelationship( "posts", [ draftPost ] );
```

New entities can also [fill relationship structs and entities](../getting-started/creating-new-entities.md#fill) without persisting the aggregate.

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

You can also use other Quick fetch methods that provide new entities if a related entity is not found, such as `firstOrNew`, `firstOrCreate`, `findOrNew`, and `findOrCreate`.

```javascript
var post = user.posts().firstOrNew( { "title": "First Post" } );
```

You can also get a new unloaded related entity by calling either the `newEntity` or the `fill` functions.

```javascript
var newPost = user.posts().fill( { "title": "My new post" } );
```

After a relationship is lazily or eagerly loaded, Quick can call a relationship-specific method and announce an interception point for each related entity. See [`quickRelationshipLoaded`](../interception-points.md#quickrelationshiploaded).

### loadRelationship

The code above is a shortcut for calling `loadRelationship` and `get{RelationName}` on an entity.

```
public any function loadRelationship( required any name, boolean force = false, boolean parallel = false )
```
