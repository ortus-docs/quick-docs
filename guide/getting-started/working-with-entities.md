# Working with Entities

## isLoaded

Checks if the entity was loaded from the database.

| Name          | Type | Required | Default | Description |
| ------------- | ---- | -------- | ------- | ----------- |
| No parameters |      |          |         |             |

A loaded entity has a tie to the database. It has either been loaded from the database or saved to the database. An unloaded entity is one created in code but not saved to the database yet.

```javascript
var user = getInstance( "User" );
user.isLoaded(); // false
user.save();
user.isLoaded(); // true
```

## clone

Clones the entity and returns an exact copy of the entity.

| Name       | Type    | Required | Default | Description                               |
| ---------- | ------- | -------- | ------- | ----------------------------------------- |
| markLoaded | boolean | false    | false   | Mark the returned entity as loaded or not |

Clones an entity and return an exact copy of the entity. This returned object is a new and separate instance from the original object.

```javascript
var user = getInstance( "User" );
var clonedUser = user.clone();
```

Unlike `replicate`, `clone` includes the primary key. The cloned entity is unloaded unless `markLoaded` is `true`.

## replicate

Creates a new, unloaded entity with the current attributes except for the primary key and any additional attributes you exclude.

| Name   | Type  | Required | Default | Description                                        |
| ------ | ----- | -------- | ------- | -------------------------------------------------- |
| except | array | `false`  | `[]`    | Additional attribute aliases or columns to omit.  |

```javascript
var original = getInstance( "Post" ).findOrFail( 1 );
var draft = original
    .replicate( [ "publishedDate" ] )
    .setTitle( "Copy of #original.getTitle()#" )
    .save();
```

Replication fires the [`postReplicate` and `quickPostReplicate` events](../interception-points.md#quickpostreplicate) with both the new `entity` and the `original` entity.

## isDirty

Returns whether the entity has changed since it was loaded or last saved. Pass an attribute alias or column name to inspect one attribute.

```javascript
var user = getInstance( "User" ).findOrFail( 1 );
user.isDirty(); // false

user.setEmail( "new@example.com" );
user.isDirty(); // true
user.isDirty( "email" ); // true
user.isDirty( "username" ); // false
```

## isClean

`isClean` is the inverse of `isDirty` and accepts the same optional attribute argument.

```javascript
user.isClean( "username" ); // true
user.isClean( "email" ); // false
```

## reset

Resets attributes to their originally loaded values and clears loaded relationships. In Quick 13, `reset()` also clears the cached query builder so old query constraints cannot leak into later calls on the entity.

```javascript
var users = getInstance( "User" );
users.whereActive( true );
users.reset();

// The previous where constraint has been cleared.
var allUsers = users.get();
```

Pass `toNew = true`, or call `resetToNew()`, to clear the attributes and mark the entity as unloaded.

## Loaded Primary Keys Are Immutable

Quick prevents changing the primary key of a loaded entity. Assigning a different key throws `QuickPrimaryKeyMutationException` so a later update cannot target one record using another record's identity.

Create a new entity or use `replicate()` when you need a different primary key.
