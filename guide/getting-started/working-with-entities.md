# Working with Entities

## isLoaded

Checks if the entity was loaded from the database.

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| No parameters |  | \`\` | \`\` |  |

A loaded entity has a tie to the database.  It has either been loaded from the database or saved to the database.  An unloaded entity is one created in code but not saved to the database yet.

```javascript
var user = getInstance( "User" );
user.isLoaded(); // false
user.save();
user.isLoaded(); // true
```

## clone

Clones the entity and returns an exact copy of the entity.

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
|  markLoaded| boolean | false | false | Mark the returned entity as loaded or not |

Clones an entity and return an exact copy of the entity. This returned object is a new and separate instance from the original object. 

```javascript
var user = getInstance( "User" );
var clonedUser = user.clone();
