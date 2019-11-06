# What's New?

## 2.4.0

* Apply custom setters when hydrating from the database.
* [Query scopes can return any value.](getting-started/query-scopes.md#scopes-that-return-values)  This allows you to use scopes to perform query functions and return values.  \(If you do not want to return a custom value, return the `QueryBuilder` instance or nothing.\)
* Improve error messages for not loaded entities.

## 2.3.0

* [Option to ignore non-existent attributes](getting-started/updating-existing-entities.md#update)

## 2.2.0

* [Relationship Fetch Methods](relationships/retrieving-relationships.md) \(`first` and `find` methods\)

## 2.1.0

* [Subselect Helper](getting-started/query-scopes.md#subselects)
* [Global Scopes](getting-started/query-scopes.md#global-scopes)
* [saveMany](relationships/relationship-types/hasmany.md#saveMany)
* Mapping foreign keys for relationships is now optional
* Either entities or primary key values can be passed to relationship persistance methods
* Relationships can also be saved by calling `"set" & relationshipName`
* [Virtual Inheritance](getting-started/defining-an-entity.md) works on ColdBox 5.2+

