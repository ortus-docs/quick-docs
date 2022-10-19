# Working with Entities

## isLoaded

Checks if the entity was loaded from the database.

| Name          | Type | Required | Default | Description |
| ------------- | ---- | -------- | ------- | ----------- |
| No parameters |      | ``       | ``      |             |

A loaded entity has a tie to the database.  It has either been loaded from the database or saved to the database.  An unloaded entity is one created in code but not saved to the database yet.

```javascript
var user = getInstance( "User" );
user.isLoaded(); // false
user.save();
user.isLoaded(); // true
```
