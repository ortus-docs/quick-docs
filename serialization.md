# Serialization

## getMemento

The memento pattern is an established pattern in ColdBox apps. A `memento` in this case is a simple representation of your entity using arrays, structs, and simple values.

For instance, the following example shows a User entity and its corresponding memento:

```javascript
component extends="quick.models.BaseEntity" {

    property name="id";
    property name="username";
    property name="email";
    property name="password";
    property name="createdDate";
    property name="modifiedDate";

}
```

```json
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "password" = "$2a$04$2nVI5rPOfl6.hrflkhBWOObO5Z7lXGJpi1vlosY74NrL/CKdpWqZS"
    "createdDate" = "{ts '2018-03-12 16:14:10'}",
    "modifiedDate" = "{ts '2018-03-12 16:14:10'}"
}
```

You can modify the memento by overriding the `getMemento` function on your entity.

```javascript
component extends="quick.models.BaseEntity" {

    property name="id";
    property name="username";
    property name="email";
    property name="password";
    property name="createdDate";
    property name="modifiedDate";

    function getMemento() {
        return {
            id = getId(),
            username = getUsername(),
            email = getEmail(),
            createdDate = dateFormat( getCreatedDate(), "MM/DD/YYYY" ),
            // can also use getAttribute if you want to bypass a custom getter
            modifiedDate = dateFormat( retrieveAttribute( "modifiedDate" ), "MM/DD/YYYY" )
        };
    }

}
```

```json
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "createdDate" = "03/12/2018",
    "modifiedDate" = "03/12/2018"
}
```

## $renderData

The `$renderData` method is a special method for ColdBox. When returning a model from a handler, this method will be called and the value returned will be used as the serialized response. This let's you simply return an entity from a handler for your API. By default this will call `getMemento()`.

```javascript
component {

    // /users/:id
    function show( event, rc, prc ) {
        return getInstance( "User" ).findOrFail( rc.id );
    }

}
```

```json
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "createdDate" = "03/12/2018",
    "modifiedDate" = "03/12/2018"
}
```

`QuickCollection` also defines a `$renderData` method, which will delegate the call to each entity in the collection and return the array of serialized entities.

```javascript
component {

    function index( event, rc, prc ) {
        return getInstance( "User" ).all();
    }

}
```

```javascript
[
    {
        "id" = 1,
        "username" = "JaneDoe",
        "email" = "jane@example.com",
        "createdDate" = "03/12/2018",
        "modifiedDate" = "03/12/2018"
    },
    {
        "id" = 2,
        "username" = "JohnDoe",
        "email" = "john@example.com",
        "createdDate" = "03/14/2018",
        "modifiedDate" = "03/15/2018"
    }
]
```

