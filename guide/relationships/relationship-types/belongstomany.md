# belongsToMany

## Usage

A `belongsToMany` relationship is a `many-to-many` relationship. For instance, a `User` may have multiple `Permissions` while a `Permission` can belong to multiple `Users`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function permissions() {
       return belongsToMany( "Permission" );
    }

}
```

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function users() {
       return belongsToMany( "User" );
    }

}
```

The first value passed to `belongsToMany` is a WireBox mapping to the related entity.

`belongsToMany` makes some assumptions about your table structure. To support a `many-to-many` relationship, you need a pivot table. This is, at its simplest, a table with each of the foreign keys as columns.

```
permissions_users
- permissionId
- userId
```

As you can see, Quick uses a convention of combining the entity table names in alphabetical order with an underscore (`_`) to create the new pivot table name. If you want to override this convention, you can do so by passing the desired table name as the second parameter or the `table` parameter.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function permissions() {
       return belongsToMany( "Permission", "user_permission_map" );
    }

}
```

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `User` entity is assumed to have a `userId` foreign key and the `Permission` entity a `permissionId` foreign key. You can override this by passing a `foreignKey` in as the third argument and a `relatedKey` as the fourth argument:

```javascript
return belongsToMany(
    "Permission",
    "user_permission_map",
    "FK_UserId",
    "FK_PermissionID"
);
```

Finally, if you are not joining on the primary keys of the current entity or the related entity, you can specify those keys using the last two parameters:

```javascript
return belongsToMany(
    "Permission",
    "user_permission_map",
    "FK_UserId",
    "FK_PermissionID",
    "user_id",
    "permission_id"
);
```

The inverse of `belongsToMany` is also `belongsToMany`. The `foreignKey` and `relatedKey` arguments are swapped on the inverse side of the relationship.

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function user() {
        belongsToMany( "User", "user_permission_map", "FK_PermissionID", "FK_UserId" );
    }

}
```

If you find yourself needing to interact with the pivot table (`permissions_users`) in the example above, you can create an intermediate entity, like `UserPermission`. You will still be able to access the end of the relationship chain using the `hasManyThrough` relationship type.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function userPermissions() {
        return hasMany( "UserPermission" );
    }

    function permissions() {
        return hasManyThrough( [ "UserPermissions", "Permission" ] );
    }

}
```

## Pivot Models

Quick hydrates the intermediate row for every related entity as a `Pivot@quick` model. Pivot key columns are always available; use `withPivot()` to include additional columns.

```javascript
function permissions() {
    return belongsToMany( "Permission" )
        .withPivot( [ "context", "active" ] );
}
```

```javascript
var permission = user.getPermissions()[ 1 ];
var pivot = permission.getPivot();

pivot.getUserId();
pivot.getPermissionId();
pivot.getContext();
```

The default pivot model is read-only. It also exposes `getPivotParent()` and `getPivotRelated()` for the two entities joined by that row.

Use `as()` to change the relationship name used for the pivot:

```javascript
function permissions() {
    return belongsToMany( "Permission" )
        .withPivot( "context" )
        .as( "assignment" );
}

var assignment = user.getPermissions()[ 1 ].getAssignment();
```

### Custom Pivot Models

Use `using()` when the pivot row needs casts, custom behavior, or persistence. The custom model must extend `quick.models.Relationships.Pivot` and declare every selected pivot column.

```javascript
// UserPermission.cfc
component
    extends="quick.models.Relationships.Pivot"
    accessors="true"
    readonly="false"
{

    property name="userId" column="user_id";
    property name="permissionId" column="permission_id";
    property name="context";
    property name="active" casts="BooleanCast@quick";

}
```

```javascript
function permissions() {
    return belongsToMany( "Permission" )
        .using( "UserPermission" )
        .withPivot( [ "context", "active" ] );
}
```

The hydrated custom pivot is a normal loaded Quick entity. Setting `readonly="false"` allows it to be updated and saved.

### Querying Pivot Columns

Pivot helpers qualify columns against the intermediate table for you:

```javascript
function activePermissions() {
    return belongsToMany( "Permission" )
        .withPivot( [ "context", "active" ] )
        .wherePivot( "active", true )
        .wherePivotNotNull( "context" )
        .orderByPivot( "context" );
}
```

The complete helper family includes:

* `wherePivot()` and `orWherePivot()`
* `wherePivotIn()` and `wherePivotNotIn()`
* `wherePivotBetween()` and `wherePivotNotBetween()`
* `wherePivotNull()` and `wherePivotNotNull()`
* `orderByPivot()` and `orderByPivotDesc()`

`withPivotValue()` both constrains the relationship and supplies a default value for future `attach()`, `sync()`, and `create()` writes.

```javascript
return belongsToMany( "Permission" )
    .withPivotValue( "active", true );
```

Use `withTimestamps()` to select and maintain pivot timestamps. The default columns are `created_at` and `updated_at`, or pass custom names.

```javascript
return belongsToMany( "Permission" )
    .withTimestamps( "created_date", "modified_date" );
```

## attach

Use the `attach` method to relate two `belongsToMany` entities together. `attach` can take a single id, a single entity, or an array of ids or entities (even mixed and matched) to associate.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );
var tag = getInstance( "Tag" ).create( { "name": "miscellaneous" });

// pass an id
post.tags().attach( tag.getId() );
// or pass an entity
post.tags().attach( tag );
```

Pass a second struct to write additional pivot attributes:

```javascript
post.tags().attach( tag, {
    "context" : "editorial",
    "active" : true
} );
```

## create

`create()` creates a new related entity and attaches it to the parent through the pivot table.

```javascript
var tag = post.tags().create(
    { "name" : "quick" },
    { "context" : "created through relationship" }
);
```

The optional arguments after the pivot attributes are `ignoreNonExistentAttributes` and query `options`, matching the related entity's normal `create()` lifecycle.

## updateExistingPivot

Update the intermediate row for one related entity without changing either side of the relationship:

```javascript
post.tags().updateExistingPivot( tag.getId(), {
    "context" : "updated",
    "active" : false
} );
```

Pivot key columns cannot be overwritten.

## detach

Use the `detach` method to remove an existing entity from a `belongsToMany` relationship. `detatch` can also take a single id, a single entity, or an array of ids or entities (even mixed and matched) to remove.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );
var tag = getInstance("Tag").firstWhere( "name", "miscellaneous" );

// pass an id
post.tags().detach( tag.getId() );
// or pass an entity
post.tags().detach( tag );
```

## sync

Sometimes you just want the related entities to be a list you give it. For these situations, use the `sync` method.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );

post.tags().sync( [ 2, 3, 6 ] );
```

Now, no matter what relationships existed before, this `Post` will only have three tags associated with it.

Like `attach()`, `sync()` accepts an optional pivot attributes struct to apply to the inserted rows.

## Relationship Setter

You can also influence the associated entities by calling `"set" & relationshipName` and passing in an entity or key value.

```javascript
var someTag = getInstance( "Tag" ).findOrFail( 2 );
var post = getInstance( "Post" ).first();
post.setTags( [ 4, 12, someTag );
```

This code calls `sync` on the relationship. After executing this code, the post would be updated in the database to be associated with the tags passed in (`4`, `12`, and `2`). Any tags that were previously associated with this post would no longer be and only the tags passed in would be associated now.

## Signature

| Name               | Type                | Required | Default                                                            | Description                                                                                                                                                                                                              |
| ------------------ | ------------------- | -------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| relationName       | string              | `true`   |                                                                    | The WireBox mapping for the related entity.                                                                                                                                                                              |
| table              | String              | `false`  | Table names in alphabetical order separated by an underscore.      | The table name used as the pivot table for the relationship. A pivot table is a table that stores, at a minimum, the primary key values of each side of the relationship as foreign keys.                                |
| foreignPivotKey    | String \| \[String] | `false`  | `keyNames()`                                                       | The name of the column on the pivot `table` that holds the value of the `parentKey` of the `parent` entity.                                                                                                              |
| relatedPivotKey    | String \| \[String] | `false`  |                                                                    | The name of the column on the pivot `table` that holds the value of the `relatedKey` of the `ralated` entity.                                                                                                            |
| parentKey          | String \| \[String] | `false`  |                                                                    | The name of the column on the `parent` entity that is stored in the `foreignPivotKey` column on `table`.                                                                                                                 |
| relatedKey         | String \| \[String] | `false`  |                                                                    | The name of the column on the `related` entity that is stored in the `relatedPivotKey` column on `table`.                                                                                                                |
| relationMethodName | String              | `false`  | The method name called on the entity to produce this relationship. | <p>The method name called to retrieve this relationship. Uses a stack backtrace to determine by default.</p><p>&#x3C;b>&#x3C;/b></p><p><strong>DO NOT PASS A VALUE HERE UNLESS YOU KNOW WHAT YOU ARE DOING.</strong></p> |

Returns a BelongsToMany relationship between this entity and the entity defined by `relationName`.

## Visualizer

{% embed url="https://codesandbox.io/embed/quick-belongs-to-many-visualizer-mj09m?autoresize=1&fontsize=14&hidenavigation=1&theme=light&view=preview" %}
