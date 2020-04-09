# hasMany

## Defining

A `hasMany` relationship is a `one-to-many` relationship. For instance, a `User` may have multiple `Posts`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function posts() {
       return hasMany( "Post" );
    }

}
```

The first value passed to `hasMany` is a WireBox mapping to the related entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `Post` entity is assumed to have a `userId` foreign key. You can override this by passing a foreign key in as the second argument:

```javascript
return hasMany("Post", "FK_userID");
```

If your parent entity does not use `id` as its primary key, or you wish to join the child entity to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key.

```javascript
return hasMany("Post", "FK_userID", "relatedPostId");
```

The inverse of `hasMany` is also [`belongsTo`](belongsto.md).

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function user() {
        return belongsTo( "User" );
    }

}
```

## Inserting & Updating

There are two ways to add an entity to a `hasMany` relationship. Both mirror the [insert API](../../getting-started/creating-new-entities.md) for entities.

### save

You can call the `save` method on the relationship passing in an entity to relate.

```javascript
var post = getInstance( "Post" ).create( {
    "title" = "My Post",
    "body" = "Hello, world!"
} );

var user = getInstance( "User" ).findOrFail( 1 );

user.posts().save( post );
// OR use the keyValue
user.posts().save( post.keyValue() );
```

This will add the `User` entity's id as a foreign key in the `Post` and save the `Post` to the database.

> **Note:** the `save` method is called on the `posts` relationship, not the `getPosts` collection.

### saveMany

You can also add many entities in a `hasMany` relationship by calling `saveMany`.  This method takes an
array of key values or entities and will associate each of them with the base entity.

### create

Use the `create` method to create and save a related entity directly through the relationship.

```javascript
var user = getInstance( "User" ).findOrFail( 1 );

user.posts().create( {
    "title" = "My Post",
    "body" = "Hello, world!"
} );
```

This example will have the same effect as the previous example.

## Removing

Removing a `hasMany` relationship is handled in two ways: either by using the `dissociate` method on the [`belongsTo`](belongsto.md) side of the relationship or by deleting the [`belongsTo`](belongsto.md) side of the relationship.

## Relationship Setter

You can also influence the associated entities by calling `"set" & relationshipName` and
passing in an array of entities or key values.

```
var postA = getInstance( "Post" ).findOrFail( 2 );
user.setPosts( [ postA, 4 ] );
```

After running this code, this user would only have two posts, the posts with ids `2` and `4`.
Any other posts would now be disassociated with this user.  Likely your database will be
guarding against creating these orphan records.  Admittedly, this method is not as likely
to be used as the others, but it does exist if it solves your use case.
