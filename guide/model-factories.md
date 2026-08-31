# Testing with Model Factories

Quick includes Laravel-inspired model factories under `quick.resources.testing`. Factories provide readable test data without putting test-only behavior on production entities.

## Defining a Factory

Create factory components outside your production model folder. The conventional name is the entity mapping followed by `Factory`.

```javascript
// tests/resources/factories/UserFactory.cfc
component extends="quick.resources.testing.Factory" {

    struct function definition() {
        return {
            "username" : "user-#lCase( createUUID() )#",
            "firstName" : "Factory",
            "lastName" : "User"
        };
    }

    any function administrator() {
        return state( { "type" : "admin" } );
    }

}
```

Every factory must implement `definition()` and return the default attribute struct for one entity.

## Factory Manager

Create a `FactoryManager` in your test base class. It resolves factory definitions and Quick entity providers through WireBox.

```javascript
variables.factoryManager = new quick.resources.testing.FactoryManager(
    wirebox = getWireBox(),
    factoryPath = "tests.resources.factories"
);

any function factory( required string name ) {
    return variables.factoryManager.factory( arguments.name );
}
```

An optional `context` struct passed to the manager is available from each definition through `getFactoryContext()`.

## Making and Creating Entities

`make()` returns an unsaved Quick entity. `create()` persists through the entity's normal `save()` lifecycle.

```javascript
var draftUser = factory( "User" ).make();
var savedUser = factory( "User" ).create();
```

Explicit attributes override the definition and any states:

```javascript
var user = factory( "User" ).create( {
    "firstName" : "Jane"
} );
```

Use `count()` to return an array of entities:

```javascript
var users = factory( "User" ).count( 3 ).create();
```

## States

States can be structs or closures. A state closure receives the attributes accumulated so far and a context containing a zero-based `index` and the requested `count`.

```javascript
var users = factory( "User" )
    .state( { "active" : true } )
    .state( function( attributes, context ) {
        return { "username" : "user-#context.index + 1#" };
    } )
    .count( 3 )
    .create();
```

Named factory methods can return `state()` and participate anywhere in a fluent chain:

```javascript
var admins = factory( "User" )
    .count( 2 )
    .administrator()
    .create();
```

## Sequences

`sequence()` cycles through an array of state structs or closures.

```javascript
var users = factory( "User" )
    .sequence( [
        { "type" : "admin" },
        { "type" : "member" }
    ] )
    .count( 4 )
    .create();
```

Attribute values may also be closures. They are evaluated after states and receive the accumulated attributes and factory context.

## Callbacks

Use `afterMaking()` and `afterCreating()` for additional test setup. Each callback receives the entity and its evaluated attributes.

```javascript
factory( "User" )
    .afterCreating( function( user, attributes ) {
        user.roles().attach( adminRole );
    } )
    .create();
```

Register callbacks for every use of a factory by overriding `configure()` in the factory definition:

```javascript
any function configure() {
    return afterMaking( function( user, attributes ) {
        user.setDisplayName( "#attributes.firstName# #attributes.lastName#" );
    } );
}
```

{% hint style="warning" %}
Factories do not manage database transactions. Integration tests should begin a transaction before each test and roll it back in `finally` so failures cannot leave factory records behind.
{% endhint %}

All implementation classes live under `resources/testing`. Production packaging can exclude that directory; Quick does not load or register factory classes during normal module startup.
