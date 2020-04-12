# Retrieving Entities

Once you have an entity and its associated database table you can start retrieving data from your database.

## Active Record

You start every interaction with Quick with an instance of an entity. The easiest way to do this is using WireBox. `getInstance` is available in all handlers by default. WireBox can easily be injected in to any other class you need using `inject="wirebox"`.

Quick is backed by qb, a CFML Query Builder. With this in mind, think of retrieving records for your entities like interacting with qb. For example:

```javascript
var users = getInstance( "User" ).all();

for ( var user in users ) {
    writeOutput( user.getUsername() );
}
```

In addition to using `for` you can utilize the `each` function on arrays. For example:

```javascript
var users = getInstance("User").all();

prc.users.each(function(user) {
    writeOutput(user.getUsername());
});
```

You can add constraints to the query just the same as you would using qb directly:

```javascript
var users = getInstance("User")
    .where("active", 1)
    .orderBy("username", "desc")
    .limit(10)
    .get();
```

> For more information on what is possible with qb, check out the [qb documentation](https://qb.ortusbooks.com).

## Quick Service

A second way to retrieve results is to use a Quick Service. It is similar to a `VirtualEntityService` from cborm.

The easiest way to create a Quick Service is to inject it using the `quickService:` dsl:

```javascript
component {
    property name="userService" inject="quickService:User"
}
```

Any method you can call on an entity can be called on the service.  A new entity will be used for all calls to a Quick Service.

```javascript
var users = userService
    .where("active", 1)
    .orderBy("username", "desc")
    .limit(10)
    .get();
```

## Aggregates

Calling qb's aggregate methods \(`count`, `max`, etc.\) will return the appropriate value instead of an entity or collection of entities.

### existsOrFail

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| id | any | `false` |  | An optional id to check if it exists. |
| errorMessage | any | `false` | `"No [#entityName()#] found with id [#arguments.id#]"` | An optional string error message or callback to produce a string error message.  If a callback is used, it is passed the unloaded entity as the only argument. |

Returns true if any entities exist with the configured query. If no entities exist, it throws an EntityNotFound exception.

```javascript
var doesUserExist = getInstance( "User" )
    .existsOrFail( rc.userID );
```

## Retrieval Methods

### all

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| No arguments |  | \`\` |  |  |

Retrieves all the records for an entity. Calling `all` will ignore any non-global constraints on the query.

```javascript
var users = getInstance( "User" ).all();
```

### get

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| No arguments |  | \`\` |  |  |

Executes the configured query, eager loads any relations, and returns the entities in a new collection.

```javascript
var posts = getInstance( "Post" )
    .whereNotNull( "publishedDate" )
    .get();
```

### paginate

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| page | numeric | `false` | 1 | The page of results to return. |
| maxRows | numeric | `false` | 25 | The number of rows to return. |

Executes the configured query, eager loads any relations, and returns the entities in the configured qb pagination struct.

```javascript
var posts = getInstance( "Post" )
    .whereNotNull( "publishedDate" )
    .paginate( rc.page, rc.maxrows );
```

```javascript
// default response example
{
    "results": [ User#1, User#2, ... ],
    "pagination": {
        "totalPages": 2,
        "maxRows": 25,
        "offset": 0,
        "page": 1,
        "totalRecords": 40
    }
}
```

### first

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| No arguments |  | \`\` |  |  |

Executes the configured query and returns the first entity found.  If no entity is found, returns `null`.

```javascript
var user = getInstance( "User" )
    .where( "username", rc.username )
    .first();
```

### firstWhere

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| column | any | `true` |  | The name of the column with which to constrain the query. A closure can be passed to begin a nested where statement. |
| operator | any | `true` |  | The operator to use for the constraint \(i.e. "=", "&lt;", "&gt;=", etc.\). A value can be passed as the `operator` and the `value` left null as a shortcut for equals \(e.g. `where( "column", 1 ) == where( "column", "=", 1 )` \). |
| value | any | `false` |  | The value with which to constrain the column. An expression \(`builder.raw()`\) can be passed as well. |
| combinator | string | `false` | `"and"` | The boolean combinator for the clause \(e.g. "and" or "or"\). Default: "and" |

Adds a basic where clause to the query and returns the first result.

```javascript
var user = getInstance( "User" )
    .firstWhere( "username", rc.username );
```

### firstOrFail

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| errorMessage | any | `false` | `"No [#entityName()#] found with constraints [#serializeJSON( retrieveQuery().getBindings() )#]"` | An optional string error message or callback to produce a string error message.  If a callback is used, it is passed the unloaded entity as the only argument. |

Executes the configured query and returns the first entity found.  If no entity is found, then an `EntityNotFound` exception is thrown with the given or default error message.

```javascript
var user = getInstance( "User" )
    .where( "username", rc.username )
    .firstOrFail();
```

### firstOrNew

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| attrs | struct | `false` | `{}` | A struct of attributes to restrict the query. If no entity is found the attrs are filled on the new entity returned. |
| newAttrs | struct | `false` | `{}` | A struct of attributes to fill on the new entity if no entity is found. These attributes are combined with `attrs`. |

Finds the first matching record or returns an unloaded new entity.

```javascript
var user = getInstance( "User" )
    .firstOrNew( { "username": rc.username } );
```

### firstOrCreate

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| attrs | struct | `false` | `{}` | A struct of attributes to restrict the query. If no entity is found the attrs are filled on the new entity created. |
| newAttrs | struct | `false` | `{}` | A struct of attributes to fill on the created entity if no entity is found. These attributes are combined with `attrs`. |

Finds the first matching record or creates a new entity.

```javascript
var user = getInstance( "User" )
    .firstOrNew( { "username": rc.username } );
```

### find

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| id | any | `true` |  | The id value to find. |

Returns the entity with the id value as the primary key. If no record is found, it returns null instead.

```javascript
var user = getInstance( "User" )
    .find( rc.userID );
```

### findOrFail

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| id | any | `true` |  | The id value to find. |
| errorMessage | any | `false` | `"No [#entityName()#] found with id [#arguments.id#]"` | An optional string error message or callback to produce a string error message.  If a callback is used, it is passed the unloaded entity as the only argument. |

Returns the entity with the id value as the primary key. If no record is found, it throws an `EntityNotFound` exception.

```javascript
var user = getInstance( "User" )
    .findOrFail( rc.userID );
```

### findOrNew

<table>
  <thead>
    <tr>
      <th style="text-align:left">Name</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Required</th>
      <th style="text-align:left">Default</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">id</td>
      <td style="text-align:left">any</td>
      <td style="text-align:left"><code>true</code>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">The id value to find.</td>
    </tr>
    <tr>
      <td style="text-align:left">attrs</td>
      <td style="text-align:left">struct</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"><code>{}</code>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>A struct of attributes to fill on the new entity if no entity is found.</p>
      </td>
    </tr>
  </tbody>
</table>Returns the entity with the id value as the primary key. If no record is found, it returns a new unloaded entity.

```javascript
var user = getInstance( "User" ).findOrNew(
    9999,
	  {
			  "firstName" : "doesnt",
			  "lastName"  : "exist"
	  }
);
```

### findOrCreate

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| id | any | `true` |  | The id value to find. |
| attrs | struct | `false` | `{}` | A struct of attributes to use when creating the new entity if no entity is found. |

Returns the entity with the id value as the primary key. If no record is found, it returns a newly created entity.

```javascript
var user = getInstance( "User" ).findOrCreate(
    9999,
	  {
        "username"  : "doesntexist",
			  "firstName" : "doesnt",
			  "lastName"  : "exist",
			  "password"  : "secret"
	  }
);
```

## Custom Collections

If you would like collections of entities to be returned as something besides an array, you can override the `newCollection` method.  It receives the array of entities.  You can return any custom collection you desire.

### newCollection

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| entities | array | `false` | `[]` | The array of entities returned by the query. |

Returns a new collection of the given entities. It is expected to override this method in your entity if you need to specify a different collection to return. You can also call this method with no arguments to get an empty collection.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function newCollection( array entities = [] ) {
        return variables._wirebox.getInstance(
            name = "extras.QuickCollection",
            initArguments = {
                "collection" = arguments.entities
            }
        );
    }

}
```

