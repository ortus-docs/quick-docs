# Creating New Entities

## save

New Quick entities can be created and persisted to the database by creating a new entity instance, setting the attributes on the entity, and then calling the `save` method.

```
var user = getInstance( "User" );
user.setUsername( "JaneDoe" );
user.setEmail( "jane@example.com" );
user.setPassword( "mypass1234" );
user.save();
```

When we call `save`, the record is persisted from the database and the primary key is set to the auto-generated value \(if any\).

## create

Another option is to use the `create` method. This method accepts a struct of data and creates a new instance with the data specified.

```
var user = getInstance( "User" ).create( {
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "password" = "mypass1234"
} );
```

