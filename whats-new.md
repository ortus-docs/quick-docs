# What's New?

## 6.1.0

### Introducing the `asQuery` method

This method is similar to `asMemento`.  You use it when the return format you want is an array of structs.  `asQuery` will skip creating entities at all, instead returning the values straight from qb.  This can result in a large performance boost depending on the number of records being returned.  Additionally, `asQuery` supports returning the columns using the alias names and returning eager loaded entities.

Read more in the [`asQuery`](guide/serialization.md#asquery) docs.

## 6.0.0

* Add compatibility with ColdBox 7.
* Fixed a bug where once a virtual attribute was added to an entity it was added to all future entities of that same type.
* Dropped support for ColdBox 5.

## 5.3.3

Use correct nesting for where statements

## 5.3.2

Fixed retrieving `null` values from fields and relationships&#x20;

## 5.3.1

* Special handling for Lucee generated keys
* Workaround for adobe@2021 not supporting null values in param statements
* Upgrade to latest cfmigrations (v4.0.0)

## 5.3.0

Add a way to not fire events inside a callback using `withoutFiringEvents`.

## 5.2.11

Upgrade to qb 9

## 5.2.10

**HasOneOrManyThrough:** Fix too many joins for `has` and `whereHas`

## 5.2.9

**HasOneOrManyThrough:** Fixed applying of through constraints\
**BaseEntity:** Use wirebox to get the util to work in CommandBox

## 5.2.8

Fix bad link to Getting Started guide

## 5.2.7

Use existing builder when adding subselects

## 5.2.6

Update to latest mementifier (v3.0.0)

## 5.2.5

CommandBox compatibility

## 5.2.4

Force generated key to be an integer.

## 5.2.3

CommandBox-friendly injections using the `box` namespace.

## 5.2.2

Fixed regression where eager loading inside a relationship was broken.

## 5.2.1

Fixed accessing QuickBuilder from entities and relationships.

## 5.2.0

Add include and exclude arguments to [fill](guide/getting-started/creating-new-entities.md#fill) method.

## 5.1.3

Always return numeric value for [`AutoIncrementingKeyType`](guide/getting-started/defining-an-entity/#primary-key).

## 5.1.2

Add the builder to the `preInsert` method for [`ReturningKeyType`](guide/getting-started/defining-an-entity/#primary-key).&#x20;

## 5.1.1

Set the table when creating new queries.

## 5.1.0

Added support for [single-table inheritance (STI)](guide/getting-started/defining-an-entity/subclass-entities.md#single-table-inheritance-sti) with discriminated subclass entities.

## 5.0.0

In 5.0.0, we stepped up the performance of Quick in a major way.  Entity creation is now over twice as fast as before.

### Performance Improvements

Average duration to create one entity:

| Engine     | 4.2.0   | 5.0.0   |
| ---------- | ------- | ------- |
| adobe@2018 | 7.65 ms | 3.71 ms |
| adobe@2021 | 6.68 ms | 2.10 ms |
| lucee@5    | 3.94 ms | 0.88 ms |

While the times may seem small, this is compounded for each entity you create.  So, if you are retrieving 1000 entities, multiply each of these numbers by 1000 and you'll start to see why this matters.

This did introduce one breaking change from v4.  See the [Upgrade Guide](upgrade-guide.md#5.0.0) for more information.

### New Features

* Added a new [`RowIDKeyType`](guide/getting-started/defining-an-entity/#primary-key) that will correctly set your primary key if your database only returns `ROWID` in the `queryExecute` response.

### Bug Fixes

* Fixed a null check in `isNullValue`.
* Fixed nulls coming back as strings in `JsonCast`.

## 4.2.0

* Add a [`simplePaginate`](guide/getting-started/retrieving-entities.md#simplepaginate) pagination method for quicker performance when total records or total pages are not needed or too slow.

## 4.1.6

* Configure columns are no longer cleared when setting up a `BelongsToMany` relationship.

## 4.1.5

* Provide `initialThroughConstraints` for `hasOne` relationships (used in `*Through` relationships).

## 4.1.4

* Ignore orders when retrieving a relationship count.

## 4.1.1, 4.1.2

* Preserve casted value after saving an entity.

## 4.1.0

* Allow for [child entities,](guide/getting-started/defining-an-entity/subclass-entities.md) including [discriminated entities.](guide/getting-started/defining-an-entity/subclass-entities.md#discriminated-entities)
* Look up returning values by column name not by alias in the `ReturningKeyType`.

## 4.0.2

* Skip eager loading database call when no keys are found.
* Only apply `CONCAT` when needed in `*Through` relationships.

## 4.0.1

* Use `WHERE EXISTS` over `DISTINCT` when fetching relationships.  `DISTINCT` restricts some of the queries that can be run.

## 4.0.0

{% hint style="success" %}
📹  [Watch a walkthrough of these changes on CFCasts.](https://cfcasts.com/series/whats-new-in-quick-4)
{% endhint %}

#### BREAKING CHANGES

* [Scopes](guide/getting-started/query-scopes-and-subselects.md), [`whereHas`](guide/relationships/querying-relationships.md#wherehas), and [`whereDoesntHave`](guide/relationships/querying-relationships.md#wheredoesnthave) callbacks now automatically group where clauses when an `OR` combinator is detected.

#### Other Changes

* Dynamically add [relationship counts](guide/relationships/relationship-counts.md) to a parent entity without loading all of the relationship.
* Give a helpful error message when trying to set relationship values before saving an entity, where applicable.
* Multiple bug fixes related to [subselects](guide/getting-started/query-scopes-and-subselects.md#subselects) and [querying relationships ](guide/relationships/querying-relationships.md)when using `belongsToThrough`, `hasOneThrough`, or `hasManyThrough`.

## 3.1.7

* Correct jQuery link in test runner.

## 3.1.6

* Allow expressions in basic where clauses.
* Fix `delete` naming collision.

## 3.1.5

* Add an alias to `with` to QuickBuilder.

## 3.1.4

* Fix a stack overflow on nested relationship checks.

## 3.1.3

* Configured tables (`.from`) are now used for qualifying columns.

## 3.1.2

* Remove unnecessary nesting in compare queries.

## 3.1.1

* Fix [querying relationship methods](guide/relationships/querying-relationships.md) when using "OR" combinators.

## 3.1.0

* Add support for JSON [casting](guide/getting-started/defining-an-entity/#casts) using a new `JsonCast@quick` component.

## 3.0.4

* Compatibility updates for ColdBox 6

## 3.0.3

* Optimize [cast ](guide/getting-started/defining-an-entity/#casts)caching

## 3.0.2

* Apply [custom sqltypes ](guide/getting-started/defining-an-entity/#sql-type)during [eager loading](guide/relationships/eager-loading.md).

## 3.0.1

* Account for null values in [`fill`](guide/getting-started/creating-new-entities.md#fill).
* Swap structAppend order for a Lucee bug in mementifier integration.

## 3.0.0

#### **BREAKING CHANGES** <a href="#breaking-changes" id="breaking-changes"></a>

_Please see the_ [_Upgrade Guide_](upgrade-guide.md#3-0-0) _for more information on these changes._

* Drop support for Lucee 4.5 and Adobe ColdFusion 11.
* Virtual Inheritance (using a `quick` annotation instead of extending `quick.models.BaseEntity`) has been removed.  It was hardly used, and removing it allows us to simplify some of the code paths.
* `accessors="true"` is now required on every entity.  This is similar to above where requiring it allows us to simplify the codebase immensely.  A helpful error message will be thrown if `accessors="true"` is not present on your entity.&#x20;
* The `defaultGrammar` mapping needs to be the full WireBox mapping, including the `@qb`, if needed.
  * For instance, `MSSQLGrammar` would become `MSSQLGrammar@qb`.
  * This will allow for other grammars to be more easily contributed via third party modules.
* [`HasManyThrough` relationships](guide/relationships/relationship-types/hasmanythrough.md) now only accept a `relationships` parameter of relationship methods to walk to get to the intended entity.
* Attributes using `casts="boolean"` need to be updated to [`casts="BooleanCast@quick"`](guide/getting-started/defining-an-entity/#casts).
* Some method and parameter names have been changed to support composite keys.  **The majority of changes will only affect you if you have extended base Quick components.** The full list can be found in the Upgrade Guide.

#### &#x20;**Other Changes** <a href="#other-changes" id="other-changes"></a>

* [Bundle Mementifier](guide/serialization.md) for memento transformations.
* Use [asMemento](guide/serialization.md#asmemento) to automatically convert queries to mementos.
* Automatically-generated [API docs](https://apidocs.ortussolutions.com/#/coldbox-modules/quick/).
* Add error message for defaulting key values.
* Update to [qb 7.0.0](https://qb.ortusbooks.com/).
* Add a [belongsToThrough relationship](guide/relationships/relationship-types/belongstothrough.md).
* Add a [HasOneThrough relationship](guide/relationships/relationship-types/hasonethrough.md).
* [Custom Casts](guide/getting-started/defining-an-entity/#casts) - using custom components to represent one or more attributes.
* [Ordering by relationship attributes](guide/relationships/ordering-by-relationships.md).
* [addSubselect](guide/getting-started/query-scopes-and-subselects.md#subselects) [improvements](guide/getting-started/query-scopes-and-subselects.md#using-relationships-in-subselects).
* Add a new QuickBuilder to better handle interop with qb.
* [Add `exists` and `existsOrFail` methods](guide/getting-started/retrieving-entities.md#existsorfail).
* Allow [custom](guide/getting-started/retrieving-entities.md#existsorfail) [error](guide/getting-started/retrieving-entities.md#firstorfail) [messages](guide/getting-started/retrieving-entities.md#findorfail) for `orFail` methods.
* Ensure `loadRelationship` doesn't reload existing relationships.
* Add multiple retrieve or new/create methods - [`firstWhere`](guide/getting-started/retrieving-entities.md#firstwhere), [`firstOrNew`](guide/getting-started/retrieving-entities.md#firstornew), [`firstOrCreate`](guide/getting-started/retrieving-entities.md#firstorcreate), [`findOrNew`](guide/getting-started/retrieving-entities.md#findornew), and [`findOrCreate`](guide/getting-started/retrieving-entities.md#findorcreate).
* Add [`paginate`](guide/getting-started/retrieving-entities.md#paginate) to Quick.
* Add [`is` and `isNot`](guide/getting-started/defining-an-entity/#comparing-entities) to compare entities.
* Allow [hydrating entities](guide/getting-started/retrieving-entities.md#hydrate) from serialized data.
* Allow returning default entities for null relations on [`HasOne`](guide/relationships/relationship-types/hasone.md#withdefault), [`BelongsTo`](guide/relationships/relationship-types/belongsto.md#withdefault), [`HasOneThrough`](guide/relationships/relationship-types/hasonethrough.md#withdefault), and [`BelongsToThrough`](guide/relationships/relationship-types/belongstothrough.md#withdefault) relationships.
* Query relations using [`has`](guide/relationships/querying-relationships.md#has), [`doesntHave`](guide/relationships/querying-relationships.md#doesnthave), [`whereHas`](guide/relationships/querying-relationships.md#wherehas), and [`whereDoesntHave`](guide/relationships/querying-relationships.md#wheredoesnthave).
* Split `reset` into `reset` and `resetToNew` methods.
* Store the original attributes for later resetting.
* Use `parameterLimits` to eager load.
* Use a new entity each time on BaseService.
* Apply sql types for columns to `wheres`.
* Apply global scopes more consistently
* Correctly ignore key column when updating.
* Fix `hasRelationship` method to only return true for exact matches.
* Better handling of constrained relationships when eager loading.
* Convert aliases when qualifying columns.
* Add a better error message if `onMissingMethod` fails.
* Only retrieve columns for defined attributes.
* Cache entity metadata in CacheBox.
* Use attribute hash for checking `isDirty`.

## 2.5.0

* Define [custom collections](guide/collections.md) per entity.

## 2.4.0

* ~~Apply custom setters when hydrating from the database.~~ (Reverted in `2.5.3` for unintended consequences with things like password hashing.)
* [Query scopes can return any value.](guide/getting-started/query-scopes-and-subselects.md#scopes-that-return-values)  This allows you to use scopes to perform query functions and return values.  (If you do not want to return a custom value, return the `QueryBuilder` instance or nothing.)
* Improve error messages for not loaded entities.
* Return the correct memento with accessors on.

## 2.3.0

* [Option to ignore non-existent attributes](guide/getting-started/updating-existing-entities.md#update)

## 2.2.0

* [Relationship Fetch Methods](guide/relationships/retrieving-relationships.md) (`first` and `find` methods)

## 2.1.0

* [Subselect Helper](guide/getting-started/query-scopes-and-subselects.md#subselects)
* [Global Scopes](guide/getting-started/query-scopes-and-subselects.md#global-scopes)
* [saveMany](guide/relationships/relationship-types/hasmany.md#saveMany)
* Mapping foreign keys for relationships is now optional
* Either entities or primary key values can be passed to relationship persistance methods
* Relationships can also be saved by calling `"set" & relationshipName`
* [Virtual Inheritance](guide/getting-started/defining-an-entity/) works on ColdBox 5.2+
