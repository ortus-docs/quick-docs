# Deleting Entities

## delete

You can delete an entity by calling the `delete` method on it.

```javascript
var user = getInstance( "User" ).find( 1 );
user.delete();
```

{% hint style="danger" %}
After a permanent delete, the entity object may still exist in variables even though its database row is gone.
{% endhint %}

## Soft Deletes

Soft deletes replace permanent deletion with a timestamp and automatically exclude deleted rows from normal queries. Enable them with `softDeletes="true"` and declare the timestamp attribute. The default attribute name is `deletedDate`.

```javascript
component
    extends="quick.models.BaseEntity"
    accessors="true"
    softDeletes="true"
{

    property name="id";
    property name="deletedDate" column="deleted_date";

}
```

Use `softDeleteColumn` component metadata when your attribute has another name:

```javascript
component
    extends="quick.models.BaseEntity"
    accessors="true"
    softDeletes="true"
    softDeleteColumn="archivedAt"
{

    property name="archivedAt" column="archived_at";

}
```

Calling `delete()` on a soft-deleting entity updates that timestamp and leaves the entity loaded.

```javascript
var user = getInstance( "User" ).findOrFail( 1 );
user.delete();

user.isTrashed(); // true
```

Normal queries exclude trashed entities. Use `withTrashed()` to include them or `onlyTrashed()` to return only deleted rows.

```javascript
var allUsers = getInstance( "User" ).withTrashed().get();
var deletedUsers = getInstance( "User" ).onlyTrashed().get();
```

Restore a loaded entity by clearing its deletion timestamp:

```javascript
var user = getInstance( "User" )
    .withTrashed()
    .findOrFail( 1 );

user.restore();
```

`restore()` uses the normal update lifecycle. Use `forceDelete()` when the row must be permanently removed.

## deleteAll

Just like `updateAll`, you can delete many records from the database by specifying a query with constraints and then calling the `deleteAll` method.

| Name | Type  | Required | Default | Description                                                                                                                                    |
| ---- | ----- | -------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| ids  | array | `false`  | `[]`    | An optional array of ids to add to the previously configured query.  The ids will be added to a WHERE IN statement on the primary key columns. |

Deletes matching entities according to the configured query.

```javascript
getInstance( "User" )
    .whereActive( false )
    .deleteAll();
```

Additionally, you can pass in an array of ids to `deleteAll` to delete only those ids.  Note that any previously configured constraints will still apply.

```javascript
getInstance( "User" ).deleteAll( [ 4, 10, 22 ] );
```

For soft-deleting entities, `deleteAll()` timestamps matching rows instead of removing them. Bulk restore and permanent deletion are also available:

```javascript
getInstance( "User" )
    .onlyTrashed()
    .whereIn( "id", [ 4, 10, 22 ] )
    .restoreAll();

getInstance( "User" )
    .whereInactive( true )
    .forceDeleteAll();
```

`forceDeleteAll()` also accepts an optional array of ids. Existing query constraints still apply.
