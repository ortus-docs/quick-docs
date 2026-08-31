# Updating Existing Entities

## save

Updates are handled identically to inserts when using the `save` method. The only difference is that instead of starting with a new entity, we start with an existing entity.

```javascript
var user = getInstance( "User" ).find( 1 );
user.setPassword( "newpassword" );
user.save();
```

`save()` applies [automatic timestamps](automatic-timestamps.md), refreshes attributes marked [`refreshOnSave="true"`](defining-an-entity/README.md#refresh-on-save), and synchronizes the entity's original state. Use [`isDirty()` and `isClean()`](working-with-entities.md#isdirty) when you need to inspect changes before saving.

## update

You can update multiple fields at once using the `update` method. This is similar to the `create` method for creating new entities.

| Name                        | Type    | Required | Default | Description                                                                                                         |
| --------------------------- | ------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------- |
| attributes                  | struct  | `false`  | `{}`    | A struct of key / value pairs to update on the entity.                                                              |
| ignoreNonExistentAttributes | boolean | `false`  | `false` | If true, does not throw an exception if an attribute does not exist.  Instead, it skips the non-existent attribute. |

```javascript
var user = getInstance( "User" ).find( 1 );
user.update( {
   email = "janedoe2@example.com",
   password = "newpassword"
} );
```

There is no need to call `save` when using the `update` method.

By default, if you have a key in the struct that doesn't match a property in the entity the `update` method will fail. If you add the optional argument `ignoreNonExistentAttributes` set to `true`, those missing keys are ignored. Now you can pass the `rc` scope from your submitted form directly into the `update` method and not worry about any other keys in the `rc` like `event` that would cause the method to fail.

```javascript
var user = getInstance( "User" ).find( 1 );
user.update( rc, true );
```

## updateOrCreate

| Name                        | Type    | Required | Default | Description                                                                                                              |
| --------------------------- | ------- | -------- | ------- | ------------------------------------------------------------------------------------------------------------------------ |
| attributes                  | struct  | `false`  | `{}`    | A struct of attributes to restrict the query. If no entity is found the attributes are filled on the new entity created. |
| newAttributes               | struct  | `false`  | `{}`    | A struct of attributes to update on the found entity or the new entity if no entity is found.                            |
| ignoreNonExistentAttributes | boolean | `false`  | `false` | If true, does not throw an exception if an attribute does not exist.  Instead, it skips the non-existent attribute.      |

Updates an existing record or creates a new record with the given attributes.

```javascript
var user = getInstance( "User" ).updateOrCreate( {
    "username": "newuser"
} );
```

## updateAll

| Name       | Type    | Required | Default | Description                                                     |
| ---------- | ------- | -------- | ------- | --------------------------------------------------------------- |
| attributes | struct  | `false`  | `{}`    | The attributes to update on the matching records.               |
| force      | boolean | `false`  | `false` | If true, skips read-only entity and read-only attribute checks. |

Updates matching entities with the given attributes according to the configured query. This is analagous to [qb's update method.](https://qb.ortusbooks.com/query-builder/executing-queries/inserts-updates-deletes#update)

```javascript
getInstance( "User" )
    .where( "lastLoggedIn", ">", dateAdd( "m", 3, now() ) )
    .updateAll( {
        "active" = 0
} );
```

When automatic timestamps are enabled, `updateAll()` also updates the configured modification timestamp. Use [`withoutAutomaticTimestamps()`](automatic-timestamps.md#disabling-automatic-timestamps) for a bulk update that must leave it unchanged.

## upsert

`upsert()` inserts rows that do not exist and updates rows matching the target columns. It exposes [qb 14's cross-database upsert support](https://qb.ortusbooks.com/query-builder/executing-queries/inserts-updates-deletes#upsert) while applying Quick attribute aliases, SQL types, read-only guards, and automatic timestamps.

```javascript
getInstance( "User" ).upsert(
    values = [
        { "email" : "jane@example.com", "username" : "Jane" },
        { "email" : "john@example.com", "username" : "John" }
    ],
    target = [ "email" ],
    update = [ "username" ]
);
```

Like `updateAll()`, this is a bulk mutation. It does not hydrate entities or fire per-entity lifecycle events.

The full signature also accepts qb's `source`, `deleteUnmatched`, `options`, `toSql`, and `matchNulls` arguments. Pass `force = true` as the final Quick-specific argument to skip read-only entity and attribute checks.

## touch

`touch()` updates an entity's configured timestamp fields without changing its in-memory state. See [Automatic Timestamps](automatic-timestamps.md#touch) for configuration and examples.

## fresh

| Name         | Type | Required | Default | Description |
| ------------ | ---- | -------- | ------- | ----------- |
| No arguments |      | \`\`     |         |             |

Retrieves a new entity from the database with the same key value as the current entity. Useful for seeing any changes made to the record in the database. This function executes a query.

`fresh()` replays the query used to load the entity, preserving scoped virtual projections and subselects.

```javascript
var user = getInstance( "User" ).findOrFail( rc.userID );
var sameUser = user.fresh();
```

## refresh

| Name         | Type | Required | Default | Description |
| ------------ | ---- | -------- | ------- | ----------- |
| No arguments |      | \`\`     |         |             |

Refreshes the attributes data for the entity with data from the database. This differs from `fresh` in that it operates on the current entity instead of returning a new one. This function executes a query.

Like `fresh()`, `refresh()` preserves scoped virtual projections from the original query.

```javascript
var user = getInstance( "User" ).findOrFail( rc.userID );
user.refresh(); // user now has updated data from the database
```
