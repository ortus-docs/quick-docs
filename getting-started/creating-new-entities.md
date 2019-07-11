# Creating New Entities

## save

New Quick entities can be created and persisted to the database by creating a new entity instance, setting the attributes on the entity, and then calling the `save` method.

```javascript
var user = getInstance( "User" );
user.setUsername( "JaneDoe" );
user.setEmail( "jane@example.com" );
user.setPassword( "mypass1234" );
user.save();
```

When we call `save`, the record is persisted from the database and the primary key is set to the auto-generated value \(if any\).

## create

Another option is to use the `create` method. This method accepts a struct of data and creates a new instance with the data specified.

```javascript
var user = getInstance( "User" ).create( {
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "password" = "mypass1234"
} );
```

By default, if you have a key in the struct that doesn't match a property in the entity the `create` method will fail. If you add the optional argument `ignoreNonExistentAttributes` set to `true`, those missing keys are ignored. Now you can pass the `rc` scope from your submitted form directly into the `create` method and not worry about any other keys in the `rc` like `event` that would cause the method to fail.

```javascript
var user = getInstance( "User" ).create( rc, true );
```
