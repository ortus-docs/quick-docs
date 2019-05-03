# belongsTo

A `belongsTo` relationship is a `many-to-one` relationship. For instance, a `Post` may belong to a `User`.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function user() {
       return belongsTo( "User" );
    }

}
```

The first value passed to `belongsTo` is a WireBox mapping to the related entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `Post` entity is assumed to have a `userId` foreign key. You can override this by passing a foreign key in as the second argument:

```javascript
return belongsTo("User", "FK_userID");
```

If your parent entity does not use `id` as its primary key, or you wish to join the child entity to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key.

```javascript
return belongsTo("User", "FK_userID", "relatedPostId");
```

The inverse of `belongsTo` is [`hasMany`](hasmany.md) or [`hasOne`](hasone.md).

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasMany( "Post" );
    }

    function latestPost() {
        // remember, relationships are just queries!
        return hasOne( "Post" ).orderBy( "createdDate", "desc" );
    }

}
```

## Updating

To update a `belongsTo` relationship, use the `associate` method. `associate` takes the entity to associate as the only argument.

```javascript
var post = getInstance("Post").findOrFail(1);

var user = getInstance("User").findOrFail(1);

post.user().associate(user);

post.save();
```

> **Note:** `associate` does not automatically save the entity. Make sure to call `save` when you are ready to persist your changes to the database.

## Removing

To remove a `belongsTo` relationship, use the `dissociate` method.

```javascript
var post = getInstance("Post").findOrFail(1);

post.user().dissociate();

post.save();
```

> **Note:** `dissociate` does not automatically save the entity. Make sure to call `save` when you are ready to persist your changes to the database.

## Relationship Setter

You can also influence the associated entities by calling `"set" & relationshipName` and
passing in an entity or key value.

```js
var post = getInstance( "Post" ).first();
post.setAuthor( 1 );
```

After executing this code, the post would be updated in the database to be associated with
the user with an id of `1`.
