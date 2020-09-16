# belongsToMany

## Usage

A `belongsToMany` relationship is a `many-to-many` relationship. For instance, a `User` may have multiple `Permissions` while a `Permission` can belong to multiple `Users`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function permissions() {
       return belongsToMany( "Permission" );
    }

}
```

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function users() {
       return belongsToMany( "User" );
    }

}
```

The first value passed to `belongsToMany` is a WireBox mapping to the related entity.

`belongsToMany` makes some assumptions about your table structure. To support a `many-to-many` relationship, you need a pivot table. This is, at its simplest, a table with each of the foreign keys as columns.

```text
permissions_users
- permissionId
- userId
```

As you can see, Quick uses a convention of combining the entity table names in alphabetical order with an underscore \(`_`\) to create the new pivot table name. If you want to override this convention, you can do so by passing the desired table name as the second parameter or the `table` parameter.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function permissions() {
       return belongsToMany( "Permission", "user_permission_map" );
    }

}
```

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `User` entity is assumed to have a `userId` foreign key and the `Permission` entity a `permissionId` foreign key. You can override this by passing a `foreignKey` in as the third argument and a `relatedKey` as the fourth argument:

```javascript
return belongsToMany(
    "Permission",
    "user_permission_map",
    "FK_UserId",
    "FK_PermissionID"
);
```

Finally, if you are not joining on the primary keys of the current entity or the related entity, you can specify those keys using the last two parameters:

```javascript
return belongsToMany(
    "Permission",
    "user_permission_map",
    "FK_UserId",
    "FK_PermissionID",
    "user_id",
    "permission_id"
);
```

The inverse of `belongsToMany` is also `belongsToMany`. The `foreignKey` and `relatedKey` arguments are swapped on the inverse side of the relationship.

```javascript
// Permission.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function user() {
        belongsToMany( "User", "user_permission_map", "FK_PermissionID", "FK_UserId" );
    }

}
```

If you find yourself needing to interact with the pivot table \(`permissions_users`\) in the example above, you can create an intermediate entity, like `UserPermission`. You will still be able to access the end of the relationship chain using the `hasManyThrough` relationship type.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function userPermissions() {
        return hasMany( "UserPermission" );
    }

    function permissions() {
        return hasManyThrough( [ "UserPermissions", "Permission" ] );
    }

}
```

## attach

Use the `attach` method to relate two `belongsToMany` entities together. `attach` can take a single id, a single entity, or an array of ids or entities \(even mixed and matched\) to associate.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );
var tag = getInstance( "Tag" ).create( { "name": "miscellaneous" });

// pass an id
post.tags().attach( tag.getId() );
// or pass an entity
post.tags().attach( tag );
```

## detach

Use the `detach` method to remove an existing entity from a `belongsToMany` relationship. `detatch` can also take a single id, a single entity, or an array of ids or entities \(even mixed and matched\) to remove.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );
var tag = getInstance("Tag").firstWhere( "name", "miscellaneous" );

// pass an id
post.tags().detach( tag.getId() );
// or pass an entity
post.tags().detach( tag );
```

## sync

Sometimes you just want the related entities to be a list you give it. For these situations, use the `sync` method.

```javascript
var post = getInstance( "Post" ).findOrFail( 1 );

post.tags().sync( [ 2, 3, 6 ] );
```

Now, no matter what relationships existed before, this `Post` will only have three tags associated with it.

## Relationship Setter

You can also influence the associated entities by calling `"set" & relationshipName` and passing in an entity or key value.

```javascript
var someTag = getInstance( "Tag" ).findOrFail( 2 );
var post = getInstance( "Post" ).first();
post.setTags( [ 4, 12, someTag );
```

This code calls `sync` on the relationship. After executing this code, the post would be updated in the database to be associated with the tags passed in \(`4`, `12`, and `2`\). Any tags that were previously associated with this post would no longer be and only the tags passed in would be associated now.

## Signature

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
      <td style="text-align:left">relationName</td>
      <td style="text-align:left">string</td>
      <td style="text-align:left"><code>true</code>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">The WireBox mapping for the related entity.</td>
    </tr>
    <tr>
      <td style="text-align:left">table</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left">Table names in alphabetical order separated by an underscore.</td>
      <td
      style="text-align:left">
        <p></p>
        <p>The table name used as the pivot table for the relationship. A pivot table
          is a table that stores, at a minimum, the primary key values of each side
          of the relationship as foreign keys.</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">foreignPivotKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"><code>keyNames()</code>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>The name of the column on the pivot <code>table</code> that holds the value
          of the <code>parentKey</code> of the <code>parent</code> entity.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">relatedPivotKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p></p>
        <p>The name of the column on the pivot <code>table</code> that holds the value
          of the <code>relatedKey</code> of the <code>ralated</code> entity.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">parentKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p></p>
        <p>The name of the column on the <code>parent</code> entity that is stored
          in the <code>foreignPivotKey</code> column on <code>table</code>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">relatedKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">
        <p></p>
        <p>The name of the column on the <code>related</code> entity that is stored
          in the <code>relatedPivotKey</code> column on <code>table</code>.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">relationMethodName</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left">The method name called on the entity to produce this relationship.</td>
      <td
      style="text-align:left">
        <p>The method name called to retrieve this relationship. Uses a stack backtrace
          to determine by default.</p>
        <p>&lt;b&gt;&lt;/b&gt;</p>
        <p><b>DO NOT PASS A VALUE HERE UNLESS YOU KNOW WHAT YOU ARE DOING.</b>
        </p>
        </td>
    </tr>
  </tbody>
</table>

Returns a BelongsToMany relationship between this entity and the entity defined by `relationName`.

## Visualizer

{% embed url="https://codesandbox.io/embed/quick-belongs-to-many-visualizer-mj09m?autoresize=1&fontsize=14&hidenavigation=1&theme=light&view=preview" %}



