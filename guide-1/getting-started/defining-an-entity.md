# Defining An Entity

To get started with Quick, you need an entity. You start by extending `quick.models.BaseEntity`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {}
```

That's all that is needed to get started with Quick. There are a few defaults of Quick worth mentioning here.

## Tables

We don't need to tell Quick what table name to use for our entity. By default, Quick uses the pluralized name of the component for the table name. That means for our `User` entity Quick will assume the table name is `users`. You can override this by specifying a `table` metadata attribute on the component.

```javascript
// User.cfc
component table="my_users" extends="quick.models.BaseEntity" accessors="true" {}
```

## Primary Key

By default, Quick assumes a primary key of `id`. The name of this key can be configured by setting `variables._key` in your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    variables._key = "user_id";

}
```

Quick also assumes a key type that is auto-incrementing. If you would like a different key type, override the`keyType` function and return the desired key type from that function.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function keyType() {
        return variables._wirebox.getInstance( "UUIDKeyType@quick" );
    }

}
```

Quick ships with the following key types:

* `AutoIncrementingKeyType`
* `NullKeyType`
* `ReturningKeyType`
* `UUIDKeyType`

`keyType` can be any component that adheres to the `keyType` interface, so feel free to create your own and distribute them via ForgeBox.

```javascript
interface displayname="KeyType" {

    /**
    * Called to handle any tasks before inserting into the database.
    * Recieves the entity as the only argument.
    */
    public void function preInsert( required entity );

    /**
    * Called to handle any tasks after inserting into the database.
    * Recieves the entity and the queryExecute result as arguments.
    */
    public void function postInsert( required entity, required struct result );

}
```

## Attributes

You specify what attributes are retrieved by adding properties to your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="username";
    property name="email";

}
```

Now, only the `id`, `username`, and `email` attributes will be retrieved.

{% hint style="warning" %}
Make sure to include the primary key \(`id` by default\) as a property.
{% endhint %}

### Persistent

To prevent Quick from mapping a property to a database column add the `persistent="false"` attribute to the property.  This is needed mostly when using dependency injection.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    property name="bcrypt" inject="@BCrypt" persistent="false";

    property name="id";
    property name="username";
    property name="email";

}
```

### Column

If the column name in your table is not the column name you wish to use in Quick, you can specify the column name using the `column` metadata attribute.  The attribute will be available using the `name` of the attribute.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="username" column="user_name";
    property name="countryId" column="FK_country_id";

}
```

### Null Values

To work around CFML's lack of `null`, you can use the `nullValue` and `convertToNull` attributes.

`nullValue` defines the value that is considered `null` for a attribute.  By default it is an empty string. \(`""`\)

`convertToNull` is a flag that, when false, will not try to insert `null` in to the database.  By default this flag is `true`.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="number" convertToNull="false";
    property name="title" nullValue="REALLY_NULL";

}
```

### Read Only

The `readOnly` attribute will prevent setters, updates, and inserts to a attribute when set to `true`.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="createdDate" readonly="true";

}
```

### SQL Type

In some cases you will need to specify an exact SQL type for your attribute.  Any value set for the `sqltype` attribute will be used when inserting or updating the attribute in the database.  It will also be used when you use the attribute in a `where` constraint.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="number" sqltype="cf_sql_varchar";

}
```

### Casts

The `casts` attribute allows you to use a value in your CFML code as a certain type while being a different type in the database.  A common example of this is a `boolean` which is usually represented as a `BIT` in the database.

The `casts` attribute must point to a WireBox mapping that resolves to a component that implements the `quick.models.Casts.CastsAttribute` interface. \(The `implements` keyword is optional.\)  This component defines how to `get` a value from the database in to the casted value and how to `set` a casted value back to the database.  Below is an example of the built-in `BooleanCast`, which comes bundled with Quick.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="active" casts="BooleanCast@quick";

}
```

```javascript
// BooleanCast.cfc
component implements="CastsAttribute" {

	/**
	 * Casts the given value from the database to the target cast type.
	 *
	 * @entity      The entity with the attribute being casted.
	 * @key         The attribute alias name.
	 * @value       The value of the attribute.
	 *
	 * @return      The casted attribute.
	 */
	public any function get(
		required any entity,
		required string key,
		any value
	) {
		return isNull( arguments.value ) ? false : booleanFormat( arguments.value );
	}

	/**
	 * Returns the value to assign to the key before saving to the database.
	 *
	 * @entity      The entity with the attribute being casted.
	 * @key         The attribute alias name.
	 * @value       The value of the attribute.
	 *
	 * @return      The value to save to the database. A struct of values
	 *              can be returned if the cast value affects multiple attributes.
	 */
	public any function set(
		required any entity,
		required string key,
		any value
	) {
		return arguments.value ? 1 : 0;
	}

}

```

Casted values are lazily loaded and cached for the lifecycle of the component.  Only cast values that have been loaded will have `set` called on them when persisting to the database.

Casts can be composed of multiple fields as well.  Take this `Address` value object, for example:

```javascript
// Address.cfc
component accessors="true" {

    property name="streetOne";
    property name="streetTwo";
    property name="city";
    property name="state";
    property name="zip";

    function fullStreet() {
        var street = [ getStreetOne(), getStreetTwo() ];
        return street.filter( function( part ) {
            return !isNull( part ) && part != "";
        } ).toList( chr( 10 ) );
    }

    function formatted() {
        return fullStreet() & chr( 10 ) & "#getCity()#, #getState()# #getZip()#";
    }

}
```

This component is not a Quick entity.  Instead it represents a combination of fields stored on our `User` entity:

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="username";
    property name="firstName" column="first_name";
    property name="lastName" column="last_name";
    property name="password";

    property name="address"
        casts="AddressCast"
        persistent="false"
        getter="false"
        setter="false";
    property name="streetOne";
    property name="streetTwo";
    property name="city";
    property name="state";
    property name="zip";
    
}
```

Noticed that the casted `address` is neither `persistent` nor does it have a `getter` or `setter` created for it.

The last piece of the puzzle is our `AddressCast` component that handles casting the value to and from the native database values:

```javascript
component implements="quick.models.Casts.CastsAttribute" {

    property name="wirebox" inject="wirebox";

    /**
     * Casts the given value from the database to the target cast type.
     *
     * @entity      The entity with the attribute being casted.
     * @key         The attribute alias name.
     * @value       The value of the attribute.
     * @attributes  The struct of attributes for the entity.
     *
     * @return      The casted attribute.
     */
    public any function get(
        required any entity,
        required string key,
        any value
    ) {
        return wirebox.getInstance( dsl = "Address" )
            .setStreetOne( entity.retrieveAttribute( "streetOne" ) )
            .setStreetTwo( entity.retrieveAttribute( "streetTwo" ) )
            .setCity( entity.retrieveAttribute( "city" ) )
            .setState( entity.retrieveAttribute( "state" ) )
            .setZip( entity.retrieveAttribute( "zip" ) );
    }

    /**
     * Returns the value to assign to the key before saving to the database.
     *
     * @entity      The entity with the attribute being casted.
     * @key         The attribute alias name.
     * @value       The value of the attribute.
     * @attributes  The struct of attributes for the entity.
     *
     * @return      The value to save to the database. A struct of values
     *              can be returned if the cast value affects multiple attributes.
     */
    public any function set(
        required any entity,
        required string key,
        any value
    ) {
        return {
            "streetOne": arguments.value.getStreetOne(),
            "streetTwo": arguments.value.getStreetTwo(),
            "city": arguments.value.getCity(),
            "state": arguments.value.getState(),
            "zip": arguments.value.getZip()
        };
    }

}
```

You can see that returning a struct of values from the `set` function assigns multiple attributes from a single cast.

### Insert & Update

You can prevent inserting and updating a property by setting the `insert` or `update` attribute to `false`.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="id";
    property name="email" column="email" update="false" insert="true";

}
```

## Formula, Computed, or Subselect properties

Quick handles formula, computed, or subselect properties using query scopes and the `addSubselect` helper method. [Check out the docs in query scopes to learn more.](query-scopes-and-subselects.md#subselects)

## Multiple datasource support

Quick uses a default datasource and default grammar, as described [here](./). If you are using multiple datasources you can override default datasource by specifying a `datasource` metadata attribute on the component. If your extra datasource has a different grammar you can override your grammar as well by specifying a `grammar` attribute.

```javascript
// User.cfc
component
    datasource="myOtherDatasource"
    grammar="PostgresGrammar@qb"
    extends="quick.models.BaseEntity"
    accessors="true"
{
    // ....
}
```

At the time of writing Valid grammar options are: `MySQLGrammar@qb`, `PostgresGrammar@qb`, `SqlServerGrammar@qb` and `OracleGrammar@qb`. Please check the [qb docs](https://qb.ortusbooks.com/) for additional options.

## Comparing Entities

You can compare entities using the `is` and `isNot` methods.  Each method takes another entity and returns `true` if the two objects represent the same entity.

```javascript
var userOne = getInstance( "User" ).findOrFail( 1 );
var userTwo = getInstance( "User" ).findOrFail( 1 );

userOne.is( userTwo ); // true
userOne.isNot( userTwo ); // false
```

