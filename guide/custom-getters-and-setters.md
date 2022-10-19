# Custom Getters & Setters

Sometimes you want to use a different value in your code than is stored in your database. Perhaps you want to enforce that setting a password always is hashed with BCrypt. Maybe you have a Date value object that you want wrapping each of your dates. You can accomplish this using custom getters and setters.

A custom getter or setter is simply a function in your entity.

To retrieve the attribute value fetched from the database, call `retrieveAttribute` passing in the name of the attribute.

To set an attribute for saving to the database, call `assignAttribute` passing in the name and the value.

```javascript
component extends="quick.models.BaseEntity" accessors="true" {

    property name="bcrypt" inject="@BCrypt";

    function setPassword( value ) {
        return assignAttribute( "password", bcrypt.hashPassword( value ) );
    }

    function getCreatedDate( value ) {
        return dateFormat( retrieveAttribute( "createdDate" ), "DD MMM YYYY" );
    }

}
```

{% hint style="danger" %}
Custom getters and setters with **not** be called when hydrating a model from the database. For that use case, use [`casts`](getting-started/defining-an-entity/#casts).
{% endhint %}
