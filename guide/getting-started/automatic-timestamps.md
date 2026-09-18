# Automatic Timestamps

Quick can maintain conventional creation and modification timestamps for you. Automatic timestamps are enabled by default in Quick 13.

When an entity declares `createdDate` and `modifiedDate` attributes, Quick will:

* set both attributes before inserting a new entity;
* set `modifiedDate` before updating an existing entity;
* add `modifiedDate` to `updateAll` and the update side of `upsert`; and
* add both timestamps to literal rows inserted through `upsert`.

Explicitly assigned timestamp values are preserved.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="createdDate" column="created_date";
    property name="modifiedDate" column="modified_date";

}
```

Timestamps are assigned before the `preSave`, `preInsert`, and `preUpdate` lifecycle events, so event listeners see the values that will be persisted.

## Custom Attribute Names

Use component metadata when your entity uses different attribute names:

```javascript
component
    extends="quick.models.BaseEntity"
    accessors="true"
    createdDateAttribute="createdAt"
    modifiedDateAttribute="updatedAt"
{

    property name="id";
    property name="createdAt" column="created_at";
    property name="updatedAt" column="updated_at";

}
```

Quick only writes timestamp attributes that are declared on the entity.

## Disabling Automatic Timestamps

Disable automatic timestamps for the entire application in your module settings:

```javascript
moduleSettings = {
    "quick" : {
        "automaticTimestamps" : false
    }
};
```

You can override the setting for one entity:

```javascript
component extends="quick.models.BaseEntity" accessors="true" automaticTimestamps="false" {
    // properties...
}
```

Or disable timestamps for one builder chain:

```javascript
getInstance( "User" )
    .whereId( 1 )
    .withoutAutomaticTimestamps()
    .updateAll( { "firstName" : "Jane" } );
```

## touch

`touch()` updates the attributes returned by `timestampFields()` using a separate query.

```javascript
var user = getInstance( "User" ).findOrFail( 1 );
user.touch();
```

By default, `timestampFields()` returns the declared `createdDate` and `modifiedDate` attributes. Override it to choose different fields:

```javascript
public array function timestampFields() {
    return [ "modifiedDate" ];
}
```

`touch()` does not change the entity's in-memory attributes or dirty state. Call `refresh()` when you also need the updated values on the current instance.
