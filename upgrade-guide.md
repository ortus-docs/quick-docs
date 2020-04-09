# Upgrade Guide

## 2.0.0

Quick 2.0 brings with it a lot of changes to make things more flexible and more performant. This shouldn't take too long — maybe 2-5 minutes per entity.

### Internal properties renamed

There were some common name clashes between internal Quick properties and custom attributes of your entities \(the most common being `fullName`\). All Quick internals have been obfuscated to avoid this situation. If you relied on these properties, please consult the following table below for the new property names.

{% hint style="warning" %}
If you are renaming your primary keys in your entities, you will have to change your key definition from `variables.key = "user_id";` to `variables._key` `= "user_id";` See [Defining an Entity](getting-started/defining-an-entity.md) for details.
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

```
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

The relationship methods are still named the same but some of the arguments have been changed to fix bugs and support better eager loading performance. Please [check the relationship docs](relationships/) for more details.

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

```
component {
    property name="userService" inject="quickService:User";
}
```

All methods available on the Quick entity are available on the service.

### Eager Loading

Eager loading is now supported for nested relationships using a dot-separated syntax. Additionally, constraints can be added to an eager loaded relationship. See the [docs on eager loading](relationships/eager-loading.md) for more information.

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

