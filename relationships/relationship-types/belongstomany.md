# belongsToMany

A `belongsToMany` relationship is a `many-to-many` relationship. For instance, a
`User` may have multiple `Permissions` while a `Permission` can belong to
multiple `Users`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function permissions() {
       return belongsToMany( "Permission" );
    }

}
```

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" {

    function users() {
       return belongsToMany( "User" );
    }

}
```

The first value passed to `belongsToMany` is a WireBox mapping to the related
entity.

`belongsToMany` makes some assumptions about your table structure. To support a
`many-to-many` relationship, you need a pivot table. This is, at its simplest, a
table with each of the foreign keys as columns.

```text
permissions_users
- permissionId
- userId
```

As you can see, Quick uses a convention of combining the entity table names in
alphabetical order with an underscore \(`_`\) to create the new pivot table
name. If you want to override this convention, you can do so by passing the
desired table name as the second parameter or the `table` parameter.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function permissions() {
       return belongsToMany( "Permission", "user_permission_map" );
    }

}
```

Quick determines the foreign key of the relationship based on the entity name
and key values. In this case, the `User` entity is assumed to have a `userId`
foreign key and the `Permission` entity a `permissionId` foreign key. You can
override this by passing a `foreignKey` in as the third argument and a
`relatedKey` as the fourth argument:

```javascript
return belongsToMany(
    "Permission",
    "user_permission_map",
    "FK_UserId",
    "FK_PermissionID"
);
```

Finally, if you are not joining on the primary keys of the current entity or the
related entity, you can specify those keys using the last two parameters:

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

The inverse of `belongsToMany` is also `belongsToMany`. The `foreignKey` and
`relatedKey` arguments are swapped on the inverse side of the relationship.

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" {

    function user() {
        belongsToMany( "User", "user_permission_map", "FK_PermissionID", "FK_UserId" );
    }

}
```

If you find yourself needing to interact with the pivot table
\(`permissions_users`\) in the example above, you can create an intermediate
entity, like `UserPermission`. You will still be able to access the end of the
relationship chain using the `hasManyThrough` relationship type.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function userPermissions() {
        return hasMany( "UserPermission" );
    }

    function permissions() {
        return hasManyThrough( "Permission", "UserPermission" );
    }

}
```

## attach

Use the `attach` method to relate two `belongsToMany` entities together.
`attach` can take a single id, a single entity, or an array of ids or entities
\(even mixed and matched\) to associate.

```javascript
var post = getInstance("Post").findOrFail(1);

var tag = getInstance("Tag").create("miscellaneous");

// pass an id
post.tags().attach(tag.getId());
// or pass an entity
post.tags().attach(tag);
```

## detach

Use the `detach` method to remove an existing entity from a `belongsToMany`
relationship. `detatch` can also take a single id, a single entity, or an array
of ids or entities \(even mixed and matched\) to remove.

```javascript
var post = getInstance("Post").findOrFail(1);

var tag = getInstance("Tag").create("miscellaneous");

// pass an id
post.tags().detach(tag.getId());
// or pass an entity
post.tags().detach(tag);
```

## sync

Sometimes you just want the related entities to be a list you give it. For these
situations, use the `sync` method.

```javascript
var post = getInstance("Post").findOrFail(1);

post.tags().sync([2, 3, 6]);
```

Now, no matter what relationships existed before, this `Post` will only have
three tags associated with it.
