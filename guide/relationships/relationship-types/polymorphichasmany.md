# polymorphicHasMany

## Usage

A `polymorphicHasMany` relationship is a `one-to-many` relationship. This relationship is used when an entity can belong to multiple types of entities. The classic example for this type of relationship is `Posts`, `Videos`, and `Comments`.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function comments() {
       return polymorphicHasMany( "Comment", "commentable" );
    }

}
```

```javascript
// Video.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function comments() {
        return polymorphicHasMany( "Comment", "commentable" );
    }

}
```

The first value passed to `polymophicHasMany` is a WireBox mapping to the related entity.

The second value is a `prefix` for the polymorphic type. A common convention where is to add `able` to the end of the entity name, though this is not automatically done. In our example, this prefix is `commentable`. This tells quick to look for a `commentable_type` and a `commentable_id` column in our `Comment` entity. It stores our entity's mapping as the `_type` and our entity's primary key value as the `_id`.

The inverse of `polymophicHasMany` is `polymorphicBelongsTo`.

```javascript
// Comment.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function post() {
        return polymorphicBelongsTo( "commentable" );
    }

}
```

## Signature

| Name               | Type                | Required | Default                                                            | Description                                                                                                                                                                                              |
| ------------------ | ------------------- | -------- | ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| relationName       | String              | `true`   |                                                                    | The WireBox mapping for the related entity.                                                                                                                                                              |
| name               | String              | `true`   |                                                                    | The name given to the polymorphic relationship.                                                                                                                                                          |
| type               | String              | `false`  | `name & "_type"`                                                   | The column name that defines the type of the polymorphic relationship.                                                                                                                                   |
| id                 | String              | `false`  | `name & "_id"`                                                     | The column name that defines the id of the polymorphic relationship.                                                                                                                                     |
| localKey           | String \| \[String] | `false`  | `keyNames()`                                                       | The local primary key on the parent entity.                                                                                                                                                              |
| relationMethodName | String              | `false`  | The method name called on the entity to produce this relationship. | <p>The method name called to retrieve this relationship.  Uses a stack backtrace to determine by default.</p><p></p><p><strong>DO NOT PASS A VALUE HERE UNLESS YOU KNOW WHAT YOU ARE DOING.</strong></p> |

Returns a polymorphicHasMany relationship between this entity and the entity defined by `relationName`.

## Visualizer

{% embed url="https://codesandbox.io/embed/quick-polymorphic-has-many-visualizer-8bgk3?autoresize=1&fontsize=14&hidenavigation=1&theme=light&view=preview" %}

