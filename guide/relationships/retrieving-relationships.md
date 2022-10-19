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

You can also call the other Quick fetch methods: `first`, `firstOrFail`, `find`, and `findOrFail` are all supported.  This is especially useful to constrain the entities available to a user by using the user's relationships:

```javascript
// This will only find posts the user has written.
var post = user.posts().findOrFail( rc.id );
```
