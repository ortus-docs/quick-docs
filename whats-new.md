# What's New?

## 3.0.0

#### **BREAKING CHANGES** <a id="breaking-changes"></a>

_Please see the_ [_Upgrade Guide_](upgrade-guide.md#3-0-0) _for more information on these changes._

* Drop support for Lucee 4.5 and Adobe ColdFusion 11.
* The `defaultGrammar` mapping needs to be the full WireBox mapping, including the `@qb`, if needed.
  * For instance, `MSSQLGrammar` would become `MSSQLGrammar@qb`.
  * This will allow for other grammars to be more easily contributed via third party modules.
* \`\`[`HasManyThrough` relationships](guide/relationships/relationship-types/hasmanythrough.md) now only accept a `relationships` parameter of relationship methods to walk to get to the intended entity.
* Attributes using `casts="boolean"` need to be updated to [`casts="BooleanCast@quick"`](guide/getting-started/defining-an-entity.md#casts).
* Some method and parameter names have been changed to support composite keys.  **The majority of changes will only affect you if you have extended base Quick components.** The full list can be found in the Upgrade Guide.

####  **Other Changes** <a id="other-changes"></a>

* [Bundle Mementifier](guide/serialization.md) for memento transformations.
* Use [asMemento](guide/serialization.md#asmemento) to automatically convert queries to mementos.
* Automatically-generated [API docs](https://apidocs.ortussolutions.com/#/coldbox-modules/quick/).
* Add error message for defaulting key values.
* Update to [qb 7.0.0](https://qb.ortusbooks.com/).
* Add a [belongsToThrough relationship](guide/relationships/relationship-types/belongstothrough.md).
* Add a [HasOneThrough relationship](guide/relationships/relationship-types/hasonethrough.md).
* [Custom Casts](guide/getting-started/defining-an-entity.md#casts) - using custom components to represent one or more attributes.
* [Ordering by relationship attributes](guide/relationships/ordering-by-relationship-attributes.md).
* [addSubselect](guide/getting-started/query-scopes-and-subselects.md#subselects) [improvements](guide/getting-started/query-scopes-and-subselects.md#using-relationships-in-subselects).
* Add a new QuickBuilder to better handle interop with qb.
* [Add `exists` and `existsOrFail` methods](guide/getting-started/retrieving-entities.md#existsorfail).
* Allow [custom](guide/getting-started/retrieving-entities.md#existsorfail) [error](guide/getting-started/retrieving-entities.md#firstorfail) [messages](guide/getting-started/retrieving-entities.md#findorfail) for `orFail` methods.
* Ensure `loadRelationship` doesn't reload existing relationships.
* Add multiple retrieve or new/create methods - [`firstWhere`](guide/getting-started/retrieving-entities.md#firstwhere), [`firstOrNew`](guide/getting-started/retrieving-entities.md#firstornew), [`firstOrCreate`](guide/getting-started/retrieving-entities.md#firstorcreate), [`findOrNew`](guide/getting-started/retrieving-entities.md#findornew), and [`findOrCreate`](guide/getting-started/retrieving-entities.md#findorcreate).
* Add [`paginate`](guide/getting-started/retrieving-entities.md#paginate) to Quick.
* Add [`is` and `isNot`](guide/getting-started/defining-an-entity.md#comparing-entities) to compare entities.
* Allow [hydrating entities](guide/getting-started/retrieving-entities.md#hydrate) from serialized data.
* Allow returning default entities for null relations on [`HasOne`](guide/relationships/relationship-types/hasone.md#withdefault), [`BelongsTo`](guide/relationships/relationship-types/belongsto.md#withdefault), [`HasOneThrough`](guide/relationships/relationship-types/hasonethrough.md#withdefault), and [`BelongsToThrough`](guide/relationships/relationship-types/belongstothrough.md#withdefault) relationships.
* Query relations using `has`, `doesntHave`, `whereHas`, and `whereDoesntHave`.
* Split `reset` into `reset` and `resetToNew` methods.
* Store the original attributes for later resetting.
* Use `parameterLimits` to eager load.
* Use a new entity each time on BaseService.
* Apply sql types for columns to `wheres`.
* Apply global scopes more consistently - Apply global scopes when paginating.
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

* ~~Apply custom setters when hydrating from the database.~~ \(Reverted in `2.5.3` for unintended consequences with things like password hashing.\)
* [Query scopes can return any value.](guide/getting-started/query-scopes-and-subselects.md#scopes-that-return-values)  This allows you to use scopes to perform query functions and return values.  \(If you do not want to return a custom value, return the `QueryBuilder` instance or nothing.\)
* Improve error messages for not loaded entities.
* Return the correct memento with accessors on.

## 2.3.0

* [Option to ignore non-existent attributes](guide/getting-started/updating-existing-entities.md#update)

## 2.2.0

* [Relationship Fetch Methods](guide/relationships/retrieving-relationships.md) \(`first` and `find` methods\)

## 2.1.0

* [Subselect Helper](guide/getting-started/query-scopes-and-subselects.md#subselects)
* [Global Scopes](guide/getting-started/query-scopes-and-subselects.md#global-scopes)
* [saveMany](guide/relationships/relationship-types/hasmany.md#saveMany)
* Mapping foreign keys for relationships is now optional
* Either entities or primary key values can be passed to relationship persistance methods
* Relationships can also be saved by calling `"set" & relationshipName`
* [Virtual Inheritance](guide/getting-started/defining-an-entity.md) works on ColdBox 5.2+

