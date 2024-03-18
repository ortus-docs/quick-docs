# Serialization

## getMemento

The memento pattern is an established pattern in ColdBox apps. A `memento` in this case is a simple representation of your entity using arrays, structs, and simple values.

For instance, the following example shows a User entity and its corresponding memento:

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="username";
    property name="email";
    property name="password";
    property name="createdDate";
    property name="modifiedDate";

}
```

```javascript
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "password" = "$2a$04$2nVI5rPOfl6.hrflkhBWOObO5Z7lXGJpi1vlosY74NrL/CKdpWqZS"
    "createdDate" = "{ts '2018-03-12 16:14:10'}",
    "modifiedDate" = "{ts '2018-03-12 16:14:10'}"
}
```

Quick bundles in the excellent [Mementifier](https://www.forgebox.io/view/mementifier) library to handle converting entities to mementos. This gives you excellent control over serialization using a `this.memento` struct on the entity and passing in arguments to the `getMemento` function.

### this.memento

By default, Quick includes all defined attributes as `includes`. You can change this or add other Mementifier options by defining your own `this.memento` struct on your entity. Your custom `this.memento` struct will be merged with Quick's default, so you can only define what changes you need.

Here is the default Quick memento struct. It is inside the `instanceReady()` lifecycle method in this example because `retrieveAttributeNames()` relies on the entity being wired (though not loaded); it is not otherwise necessary to put `this.memento` inside `instanceReady()`.

```javascript
function instanceReady() {
    this.memento = {
        "defaultIncludes" : retrieveAttributeNames( withVirtualAttributes = true ),
          "defaultExcludes" : [],
          "neverInclude"    : [],
          "defaults"        : {},
          "mappers"         : {},
          "trustedGetters"  : true,
          "ormAutoIncludes" : false
    };
}
```

### getMemento Arguments

You can also control the serialization of a memento at call time using Mementifier's `getMemento` arguments.

```javascript
struct function getMemento(
    includes = "", // or []
    excludes = "", // or []
    struct mappers = {},
    struct defaults = {},
    boolean ignoreDefaults = false,
    boolean trustedGetters
)
```

### Custom getMemento

If this does not give you the control you need, you can further modify the memento by overriding the `getMemento` function on your entity. In this case, a `$getMemento` function will be available which is the Mementifier function.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

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

```javascript
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "createdDate" = "03/12/2018",
    "modifiedDate" = "03/12/2018"
}
```

## asMemento

Sometimes when retrieving entities or executing a Quick query, you already know you want mementos back. You can skip the step of calling `getMemento` yourself or mapping over the array of results returned by calling `asMemento` before executing the query. `asMemento` takes the same arguments that `getMemento` does. It will pass those arguments on and convert your entities to mementos after executing the query. This works for all the query execution methods - `find`, `first`, `get`, `paginate`, etc.

```cfscript
getInstance( "User" )
    .asMemento( excludes = [ "password" ] )
    .paginate( 1, 10 );
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

```javascript
{
    "id" = 1,
    "username" = "JaneDoe",
    "email" = "jane@example.com",
    "createdDate" = "03/12/2018",
    "modifiedDate" = "03/12/2018"
}
```

`QuickCollection` also defines a `$renderData` method, which will delegate the call to each entity in the collection and return the array of serialized entities.

{% hint style="info" %}
Automatically serializing a returned collection only works when using the `QuickCollection` as your entity's `newCollection`.
{% endhint %}

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

## asQuery

Quick can skip creating entities all together and return an array of structs from qb using the `asQuery` method.

```cfscript
getInstance( "User" )
    .asQuery()
    .paginate( 1, 10 );
```

`asQuery` also supports Eager Loading.  It will add a key matching the relationship name to the returned struct.

```cfscript
getInstance( "Post" )
    .with( "author" )
    .asQuery()
    .paginate( 1, 10 );
```

```json
{
    "id": 1,
    "title": "My First Post",
    "body": "....",
    "author": {
        "id": 1,
        "firstName": "Eric",
        "lastName": "Peterson"    
    }
}
```

{% hint style="info" %}
Using `asQuery` is usually superior to dropping down qb using `retrieveQuery` since it achieves the same purpose while also including automatic aliases and eager loading.
{% endhint %}

### Signature

| Name        | Type    | Required | Default | Description                                                |
| ----------- | ------- | -------- | ------- | ---------------------------------------------------------- |
| withAliases | boolean | `false`  | `true`  | Use the aliases instead of the column names for the query. |

The `withAliases` property is recursive across eager loads.  If you ask for aliases for your initial query (which is the default), all eager loaded queries will also use their respective column aliases.

