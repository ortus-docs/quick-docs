# hasOne

## Defining

A `hasOne` relationship is a "one-to-one" relationship. For instance, a `User` entity might have an `UserProfile` entity attached to it.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function profile() {
       return hasOne( "UserProfile" );
    }

}
```

The first value passed to `hasOne` is a WireBox mapping to the related entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `UserProfile` entity is assumed to have a `userId` foreign key. You can override this by passing a foreign key in as the second argument:

```javascript
return hasOne( "UserProfile", "FK_userID" );
```

The inverse of `hasOne` is [`belongsTo`](belongsto.md). It is important to choose the right relationship for your database structure. `hasOne` assumes that the related model has the foreign key for the relationship.

```javascript
// UserProfile.cfc
component extends="quick.models.BaseEntity" {

    function user() {
        return belongsTo( "User" );
    }

}
```

