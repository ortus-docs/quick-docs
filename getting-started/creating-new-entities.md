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

## firstOrCreate

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| attrs | struct | `false` | `{}` | A struct of attributes to restrict the query. If no entity is found the attrs are filled on the new entity created. |
| newAttrs | struct | `false` | `{}` | A struct of attributes to fill on the created entity if no entity is found. These attributes are combined with `attrs`. |

Finds the first matching record or creates a new entity.

```javascript
var user = getInstance( "User" )
    .firstOrNew( { "username": rc.username } );
```

## findOrCreate

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| id | any | `true` |  | The id value to find. |
| attrs | struct | `false` | `{}` | A struct of attributes to use when creating the new entity if no entity is found. |

Returns the entity with the id value as the primary key. If no record is found, it returns a newly created entity.

```javascript
var user = getInstance( "User" ).findOrCreate(
    9999,
	  {
        "username"  : "doesntexist",
			  "firstName" : "doesnt",
			  "lastName"  : "exist",
			  "password"  : "secret"
	  }
);
```

