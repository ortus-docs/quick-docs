# polymorphicBelongsTo

A `polymorphicBelongsTo` relationship is a `many-to-one` relationship. This relationship is used when an entity can belong to multiple types of entities. The classic example for this type of relationship is `Posts`, `Videos`, and `Comments`. For instance, a `Comment` may belong to a `Post` or a `Video`.

```javascript
// Comment.cfc
component extends="quick.models.BaseEntity" {

    function post() {
        return polymorphicBelongsTo( "commentable" );
    }

}
```

The only value passed to `polymorphicBelongsTo` is a `prefix` for the polymorphic type. A common convention where is to add `able` to the end of the entity name, though this is not automatically done. In our example, this prefix is `commentable`. This tells quick to look for a `commentable_type` and a `commentable_id` column in our `Comment` entity. It stores our entity's mapping as the `_type` and our entity's primary key value as the `_id`.

When retrieving a `polymorphicBelongsTo` relationship the `_id` is used to retrieve a `_type` from the database.

The inverse of `polymorphicHasMany` is also `polymorphicHasMany`. It is important to choose the right relationship for your database structure. `hasOne` assumes that the related model has the foreign key for the relationship.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function comments() {
       return polymorphicHasMany( "Comment", "commentable" );
    }

}
```

```javascript
// Video.cfc
component extends="quick.models.BaseEntity" {

    function comments() {
        return polymorphicHasMany( "Comment", "commentable" );
    }

}
```

