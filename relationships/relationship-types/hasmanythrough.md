# hasManyThrough

A `hasManyThrough` relationship is a `many-to-many` relationship. It is used when you want to access a related entity through another entity. The most common example for this is through a pivot table. For instance, a `User` may have multiple `Permissions` via a `UserPermission` entity. This allows you to store additional data on the `UserPermission` entity, like a `createdDate` .

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function permissions() {
       return hasManyThrough( "Permission" );
    }

}
```

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" {

    function users() {
       return hasManyThrough( "User" );
    }

}
```

```javascript
// UserPermission.cfc
component extends="quick.models.BaseEntity" {

    function user() {
        return belongsTo( "User" );
    }

    function permission() {
        return belongsTo( "Permission" );
    }

}
```

The first value passed to `hasManyThrough` is a WireBox mapping to the related entity.

The second value passed is a WireBox mapping to the intermediate entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `Permission` entity is assumed to have a `permissionId` foreign key. You can override this by passing a foreign key in as the third argument:

```javascript
return hasManyThrough("Permission", "UserPermission", "FK_permissionID");
```

The `secondKey` is also determined by Quick. It is the foreign key of the current entity for the intermediate entity's table. In our example, this would be `userId`, since `User` is our entity and it is for the `UserPermissions` table. You can override this by passing in the `secondKey` as the fourth argument.

```javascript
return hasManyThrough(
    "Permission",
    "UserPermission",
    "FK_permissionID",
    "FK_userID"
);
```

Lastly, the `localKey` and `secondLocalKey` are the primary keys of the entity and the intermediate entities. Usually this is just `id`. You can override these as the fifth and sixth argument.

```text
return hasManyThrough(
    relationName = "Permission",
    intermeediate = "UserPermission",
    firstKey = "FK_permissionID", // foreign key on the UserPermission table
    secondKey = "FK_userID", // foreign key on the Permission table
    localKey = "userID", // local key on the owning entity table
    secondLocalKey = "id" // local key on the UserPermission table
);
```

The inverse of `hasManyThrough` is also `hasManyThrough`. A note that the intermediate entity would use `belongsTo` relationships to link back to each side of the `hasManyThrough` relationship. These relationships are not needed to use a `hasManyThrough` relationship.

