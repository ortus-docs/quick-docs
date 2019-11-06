# hasOne

## Defining

A `hasOne` relationship is a "one-to-one" relationship. For instance, a `User` entity might have an `UserProfile` entity attached to it.

```
// User.cfc
component extends="quick.models.BaseEntity" {

    function profile() {
       return hasOne( "UserProfile" );
    }

}
```

The first value passed to `hasOne` is a WireBox mapping to the related entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `UserProfile` entity is assumed to have a `userId` foreign key. You can override this by passing a foreign key in as the second argument:

```
return hasOne("UserProfile", "FK_userID");
```

If your parent entity does not use `id` as its primary key, or you wish to join the child entity to a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key.

```
return belongsTo("UserProfile", "FK_userID", "profile_id");
```

The inverse of `hasOne` is [`belongsTo`](belongsto.md). It is important to choose the right relationship for your database structure. `hasOne` assumes that the related model has the foreign key for the relationship.

```
// UserProfile.cfc
component extends="quick.models.BaseEntity" {

    function user() {
        return belongsTo( "User" );
    }

}
```

