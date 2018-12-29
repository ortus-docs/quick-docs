# polymorphicHasMany

A `polymorphicHasMany` relationship is a `one-to-many` relationship. This
relationship is used when an entity can belong to multiple types of entities.
The classic example for this type of relationship is `Posts`, `Videos`, and
`Comments`.

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

The first value passed to `polymophicHasMany` is a WireBox mapping to the
related entity.

The second value is a `prefix` for the polymorphic type. A common convention
where is to add `able` to the end of the entity name, though this is not
automatically done. In our example, this prefix is `commentable`. This tells
quick to look for a `commentable_type` and a `commentable_id` column in our
`Comment` entity. It stores our entity's mapping as the `_type` and our entity's
primary key value as the `_id`.

The inverse of `polymophicHasMany` is `polymorphicBelongsTo`.

```javascript
// Comment.cfc
component extends="quick.models.BaseEntity" {

    function post() {
        return polymorphicBelongsTo( "commentable" );
    }

}
```
