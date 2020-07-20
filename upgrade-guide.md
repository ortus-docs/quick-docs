# Upgrade Guide

## 3.0.0

### Adobe ColdFusion 11 and Lucee 4.5 are no longer supported

Please migrate to a supported engine.

### Virtual Inheritance support removed

Virtual Inheritance allowed you to use a `quick` annotation on your entity instead of extending `quick.models.BaseEntity`.  This was hardly used and didn't offer any benefit to extending using traditional inheritance.  Additionally, removing the support allows us to clean up the code base by removing duplicate code paths.

If any of your entities are using the `quick` annotation, instead have them `extends="quick.models.BaseEntity"`.

### \`accessors="true"\` required for all entities

From the early days of Quick, developers have wanted to have `accessors="true"` on their entities.  Because of this, Quick supported defining entities both with and without accessors.  However, just as with virtual inheritance, it created two code paths that could hide bugs and make it hard to follow the code.  In Quick 3.0.0, `accessors="true"` is required on all entities.  If it is omitted, a helpful error message is thrown to remind you.  This will help immensely in simplifying the code base.  \(In fact, just introducing this requirement helped find two bugs that were only present when using accessors.\)

Ensure all entities have `accessors="true"` in their component metadata.

### AssignedKey removed

Use `NullKeyType` instead.

### Boolean casts updates to hook in to new Cast system

Previously, the only valid cast type was `casts="boolean"`.  In introducing the new Casts system, the boolean cast was refactored to use the same system.  For this reason, any `casts="boolean"` needs to be changed to `casts="BooleanCast@quick"`

### defaultGrammar updated to be the full WireBox mapping <a id="defaultgrammar-updated-to-be-the-full-wirebox-mapping"></a>

In previous versions, the value passed to `defaultGrammar` was used to look up a mapping in the `@qb` namespace. This made it difficult to add or use grammars that weren't part of qb. \(You could get around this be registering your custom grammar in the `@qb` namespace, but doing so seemed strange.\)

To migrate this code, change your `defaultGrammar` to be the full WireBox mapping in your `moduleSettings`:

```javascript
moduleSettings = {
    "quick": {
        "defaultGrammar": "MSSQLGrammar@qb"
    }
};
```

### [HasManyThrough relationship](guide-1/relationships/relationship-types/hasmanythrough.md) re-tooled to be based on relationships

It no longer accepts any entities or columns. Rather, it accepts an array of relationships to walk "through" to end up at the desired entity.

Here's how the old syntax would be used to define the relationship between `Country` and `Post`.

{% tabs %}
{% tab title="Country.cfc" %}
```javascript
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasManyThrough( "Post", "User", "country_id", "user_id" );
    }
    
}
```
{% endtab %}
{% endtabs %}

This same relationship now needs to be defined in terms of other relationships, like so.

{% tabs %}
{% tab title="Country.cfc" %}
```javascript
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasManyThrough( [ "users", "posts" ] );
    }
    
    function users() {
        return hasMany( "User" );
    }

}
```
{% endtab %}

{% tab title="User.cfc" %}
```javascript
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasMany( "Post" );
    }

}
```
{% endtab %}
{% endtabs %}

This approach does require a relationship defined for each level, but it works up and down any number of relationships to get to your desired entity.

### \`[associate](guide-1/relationships/relationship-types/belongsto.md#updating)\` cannot be called on unloaded entities

To update the foreign key of a `belongsTo` relationship you use the `associate` method.  In the past, it was possible to associate a new, unsaved child entity to its parent using this method.

```javascript
var newChild = getInstance( "Child" );
newChild.fill( fields );
newChild.parent().associate( parent );
newChild.save();
```

In an attempt to provide more helpful error messages, this behavior is no longer possible.  You can achieve the same effect in one of two ways.

The first is to manually assign the foreign keys:

```javascript
var newChild = getInstance( "Child" );
newChild.fill( fields );
newChild.setParentKey( parent.getId() );
newChild.save();
```

While this works, it breaks the encapsulation provided by the relationship.

The second approach is to use the `hasOne` or `hasMany` side of the relationship to create the new child entity:

```javascript
var newChild = getInstance( "Child" );
newChild.fill( fields );
var parent = getInstance( "Parent" ).findOrFail( id );
parent.children().save( newChild );
```

If you have all the data handy in a struct, you can use the `create` method for a more concise syntax.

```javascript
var parent = getInstance( "Parent" ).findOrFail( id );
var newChild = parent.children().create( fields );
```

This is the recommended way of creating these components.

### assignAttributesData argument renamed to \`attributes\`

This brings the API in line with the other methods referencing attributes.

### Method changes due to compound key support

Compound key support required some method and parameter name changes.  Although the list seems extensive, you will likely not need to change anything in your code unless you have extended built-in Quick components.  \(You will see many relationship parameter name changes.  Note that the function you call to define a relationship is a function on the `BaseEntity` and has not changed its signature.\)

**BaseEntity.cfc:**

* `retrieveQualifiedKeyName : String` -&gt; `retrieveQualifiedKeyNames : [String]`
* `keyName : String` -&gt; `keyNames : [String]`
* `keyColumn : String` -&gt; `keyColumns : [String]`
* `keyValue : String` -&gt; `keyValues : [String]`

**AutoIncrementingKeyType.cfc**

* This cannot be used with composite primary keys

**BaseRelationship.cfc**

* `getKeys` now takes an array of `keys` as the second argument
* `getQualifiedLocalKey : String` -&gt; `getQualifiedLocalKeys : [String]`
* `getExistenceCompareKey : String` -&gt; `getExistenceCompareKeys : [String]`

**BelongsTo.cfc**

* `init` arguments have changed
  * `foreignKey : String` -&gt; `foreignKeys : [String]`
  * `ownerKey : String` -&gt; `ownerKeys : [String]`
* `getQualifiedLocalKey : String` -&gt; `getQualifiedLocalKeys : [String]`
* `getExistenceCompareKey : String` -&gt; `getExistenceCompareKeys : [String]`

**BelongsToMany.cfc**

* `init` arguments have changed
  * `foreignPivotKey : String` -&gt; `foreignPivotKeys : [String]`
  * `relatedPivotKey : String` -&gt; `relatedPivotKeys : [String]`
  * `parentKey : String` -&gt; `parentKeys : [String]`
  * `relatedKey : String` -&gt; `relatedKeys : [String]`
* `getQualifiedRelatedPivotKeyName : String` -&gt; `getQualifiedRelatedPivotKeyNames : [String]`
* `getQualifiedForeignPivotKeyName : String` -&gt; `getQualifiedForeignPivotKeyNames : [String]`
* `getQualifiedForeignKeyName : String` -&gt; getQualifiedForeignKeyNames : \[String\]\`

**HasManyThrough.cfc**

* This component now extends `quick.models.Relationships.HasOneOrManyThrough`
* `init` arguments are now as follows:
  * `related`: The related entity instance.
  * `relationName`: The WireBox mapping for the related entity.
  * `relationMethodName`: The method name called to retrieve this relationship.
  * `parent`: The parent entity instance for the relationship.
  * `relationships`: An array of relationships between the parent entity and the related entity.
  * `relationshipsMap`: A dictionary of relationship name to relationship component.
* The following methods no longer exist:
  * `getQualifiedFarKeyName`
  * `getQualifiedForeignKeyName`
  * `getQualifiedFirstKeyName`
  * `getQualifiedParentKeyName`

**HasOneOrMany.cfc**

* `init` arguments have changed
  * `foreignKey : String` -&gt; `foreignKeys : [String]`
  * `localKey : String` -&gt; `localKeys : [String]`
* `getParentKey : any` -&gt; `getParentKeys : [any]`
* `getQualifiedLocalKey : String` -&gt; `getQualifiedLocalKeys : [String]`
* `getQualifiedForeignKeyName : String` -&gt; `getQualifiedForeignKeyNames : [String]`

**PolymorphicBelongsTo.cfc**

* `init` arguments have changed
  * `foreignKey : String` -&gt; `foreignKeys : [String]`
  * `ownerKey : String` -&gt; `ownerKeys : [String]`

**PolymorphicHasOneOrMany.cfc**

* `init` arguments have changed
  * `id : String` -&gt; `ids : [String]`
  * `localKey : String` -&gt; `localKeys : [String]`

## 2.0.0

Quick 2.0 brings with it a lot of changes to make things more flexible and more performant. This shouldn't take too long — maybe 2-5 minutes per entity.

### Internal properties renamed

There were some common name clashes between internal Quick properties and custom attributes of your entities \(the most common being `fullName`\). All Quick internals have been obfuscated to avoid this situation. If you relied on these properties, please consult the following table below for the new property names.

{% hint style="warning" %}
If you are renaming your primary keys in your entities, you will have to change your key definition from `variables.key = "user_id";` to `variables._key` `= "user_id";` See [Defining an Entity](guide-1/getting-started/defining-an-entity.md) for details.
{% endhint %}

| Old Property Name | New Property Name |
| :--- | :--- |
| `builder` | `_builder` |
| `wirebox` | `_wirebox` |
| `str` | `_str` |
| `settings` | `_settings` |
| `validationManager` | `_validationManager` |
| `interceptorService` | `_interceptorService` |
| `keyType` | `_keyType` |
| `entityName` | `_entityName` |
| `mapping` | `_mapping` |
| `fullName` | `_fullName` |
| `table` | `_table` |
| `queryOptions` | `_queryOptions` |
| `readonly` | `_readonly` |
| `key` | `_key` |
| `attributes` | `_attributes` |
| `meta` | `_meta` |
| `nullValues` | `_nullValues` |
| `data` | `_data` |
| `originalAttributes` | `_originalAttributes` |
| `relationshipsData` | `_relationshipsData` |
| `eagerLoad` | `_eagerLoad` |
| `loaded` | `_loaded` |

Additionally, some method names have also changed to avoid clashing with automatically generated getters and setters. Please consult the table below for method changes.

| Old Method Name | New Method Name |
| :--- | :--- |
| `setDefaultProperties` | `assignDefaultProperties` |
| `getKeyValue` | `keyValue` |
| `getAttributesData` | `retrieveAttributesData` |
| `getAttributeNames` | `retrieveAttributeNames` |
| `setAttributesData` | `assignAttributesData` |
| `getColumnForAlias` | `retrieveColumnForAlias` |
| `getAliasForColumn` | `retrieveAliasForColumn` |
| `setOriginalAttributes` | `assignOriginalAttributes` |
| `getLoaded` | `isLoaded` |
| `getAttribute` | `retrieveAttribute` |
| `setAttribute` | `assignAttribute` |
| `getQuery` | `retrieveQuery` |
| `getRelationship` | `retrieveRelationship` |
| `setRelationship` | `assignRelationship` |

Lastly, the following properties and methods have been removed:

| Removed Property or Method |
| :--- |
| `relationships` |

### Key Types

#### Defining Key Types

Key Types are the way to define setting and retrieving a primary key in Quick. In Quick 1.0 these were injected in to the component. This made reusability hard for simple things like sequence names. In order to allow for more flexible key types, key types are no longer injected. Instead, they should be returned from a `keyType` method.

```text
function keyType() {
    return variables._wirebox.getInstance( "Sequence@quick" )
        .setSequenceName( "seq_users" );
}
```

The `keyType` is lazily created and cached on the component, so this is both a more flexible approach as well as being more performant. If you are injecting custom key types in your entities you will need to move them to the method syntax.

#### Changed Key Types

A few key types have been renamed and will need to be updated in your codebase:

| Old Key Type Name | New Key Type Name |
| :--- | :--- |
| `AssignedKey` | `NullKeyType` |
| `AutoIncrementing` | `AutoIncrementingKeyType` |
| `UUID` | `UUIDKeyType` |

#### New Key Types

In additional to the changes to defining key types, there is a few new key types introduced in Quick 2.0.

**ReturningKeyType**

Used with grammars that return their primary key in the query response when inserting to the database. An example of this is `NEWSEQUENTIALID` in Microsoft SQL Server.

### Scopes

The way arguments are passed to scopes have been updated to allow for default arguments. `query` is still the first argument. Other arguments will be passed in order after that. The `args` struct is no longer passed.

### Relationships

The relationship methods are still named the same but some of the arguments have been changed to fix bugs and support better eager loading performance. Please [check the relationship docs](guide-1/relationships/) for more details.

Additionally, the alternative syntax for defining relationships on a `relationships` struct has been removed. It created an unnecessary code path that had it's own share of bugs. All relationships should be defined as methods on the entity.

### Removing CFCollection

CFCollection was included in Quick 1.0 as both a way to lazily eager load a relationship and as a compatibility layer for older CF versions. The compatibility that CFCollection provides, however, comes with a performance cost. Additionally, the majority of users wanted to use plain arrays as the return format. For those reasons, arrays are now the default return format for collections. CFCollections can still be used by specifying a different return format in the module settings.

### Converting to Null

Null is a tricky thing in CFML. The same goes for interacting with nulls in a database. By default, we will support the CFML convention of using an empty string to represent null. When interacting with the database empty strings will be converted to nulls. You can adjust this behavior on the property level with two new annotations:

* `convertToNull` - Determines if the property will be automatically checked to convert to null at all. Defaults to `true`.
* `nullValue` - This is the value that is equivalent to null for this property. Defaults to an empty string.

### Returning Null instead of Unloaded Entities

In an effort to avoid dealing with CFML's version of `null`, Quick originally returned unloaded entities. You could check if an entity was loaded using the `isLoaded` method. This doesn't make as much sense as `null` however and even made it more difficult to interact with other libraries. Now Quick will return null when it encounters an empty query result either from a retrieval or from a `belongsTo` or `hasOne` relationship. Any instances that you were checking `isLoaded` should be updated. `isLoaded` will continue to exist for when you are creating a new entity not from the database.

### AutoDiscover Grammar

The default grammar for Quick is now `AutoDiscover`. This provides a better first run experience. The grammar can still be set in the `moduleSettings`.

### BaseService

As a new way to interact with Quick, you can use Quick Services to interact with your entities in a service-oriented fashion. These are equivalent to `VirtualEntityServices` in cborm.

The easiest way to use a Quick Service is to use the `quickService:` injection dsl.

```text
component {
    property name="userService" inject="quickService:User";
}
```

All methods available on the Quick entity are available on the service.

### Eager Loading

Eager loading is now supported for nested relationships using a dot-separated syntax. Additionally, constraints can be added to an eager loaded relationship. See the [docs on eager loading](guide-1/relationships/eager-loading.md) for more information.

### Column Aliases in Queries

Column aliases can now be used in queries. They will be transformed to columns before executing the query.

### Quick entities in Setters

If you pass a Quick entity to a setter method the entity's `keyValue` value will be passed.

### Update and Insert Guards

Columns can be prevented from being inserted or updated using property attributes — `insert="false"` and `update="false"`.

### cbvalidation removed as a default dependency

Quick no longer automatically validates entities before saving them. Having cbvalidation baked in made it hard to extend it. If desired, validation can be added back in using Quick's lifecycle hooks.

### `instanceReady` Lifecycle Method

Quick now announces an `instanceReady` event after the entity has gone through dependency injected and had its metadata inspected. This can be used to hook in other libraries, like `cbvalidation` and `mementifier`.

### Automatic Attribute Casting

You can automatically cast a property to a boolean value while retrieving it from the database and back to a bit value when serializing to the database by setting `casts="boolean"` on the property.

