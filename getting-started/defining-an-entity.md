# Defining An Entity

To get started with Quick, you need an entity. There are two ways to define an entity.

The first way is to extend `quick.models.BaseEntity`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {}
```

The second way, in a ColdBox application, is to annotate your component with the `quick` annotation.

```javascript
// User.cfc
component quick {}
```

This will use WireBox's [Virtual Inheritance](https://wirebox.ortusbooks.com/advanced-topics/virtual-inheritance) to accomplish the same as extending the `BaseEntity`.

That's all that is needed to get started with Quick. There are a few defaults of Quick worth mentioning here.

## Tables

We don't need to tell Quick what table name to use for our entity. By default, Quick uses the pluralized name of the component for the table name. That means for our `User` entity Quick will assume the table name is `users`. You can override this by specifying a `table` metadata attribute on the component.

```javascript
// User.cfc
component table="my_users" extends="quick.models.BaseEntity" {}
```

## Primary Key

By default, Quick assumes a primary key of `id`. The name of this key can be configured by setting `variables.key` in your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    variables.key = "user_id";

}
```

## Columns

Every column in the related database table is retrieved and available in your entity by default. This means that if our `users` table had the following columns \(`id`, `username`, `email`, `password`\) then we could access each of those as properties \(`getId()`, `getUsername()`, `getEmail()`, `getPassword()`\).

You can customize what columns are retrieved by adding properties to your component.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    property name="username";
    property name="email";

}
```

Now, only the `id`, `username`, and `password` columns will be retrieved.

> Note: the primary key \(`id` by default\) will be retrieved regardless of the properties specified.

To prevent Quick from mapping a property to the database add the `persistent="false"` attribute to the property.

```text
// User.cfc
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    property name="bcrypt" inject="@BCrypt" persistent="false";

    property name="username";
    property name="email";

}
```

If the column name in your table is not the column name you wish to use in quick, you can alias it using the `column` metadata attribute.

```javascript
component extends="quick.models.BaseEntity" {

    property name="username" column="user_name";
    property name="countryId" column="FK_country_id";
    
}
```

