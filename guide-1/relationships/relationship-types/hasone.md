# hasOne

## Usage

A `hasOne` relationship is a "one-to-one" relationship. For instance, a `User` entity might have an `UserProfile` entity attached to it.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function profile() {
       return hasOne( "UserProfile" );
    }

}
```

The first value passed to `hasOne` is a WireBox mapping to the related entity.

Quick determines the foreign key of the relationship based on the entity name and key values. In this case, the `UserProfile` entity is assumed to have a `userId` foreign key. You can override this by passing a foreign key in as the second argument:

```javascript
return hasOne( "UserProfile", "FK_userID" );
```

If your parent entity does not use `id` as its primary key, or you wish to join the child entity to a different column, you may pass a third argument to the `hasOne` method specifying your parent table's custom key.

```javascript
return hasOne( "UserProfile", "FK_userID", "profile_id" );
```

The inverse of `hasOne` is [`belongsTo`](belongsto.md). It is important to choose the right relationship for your database structure. `hasOne` assumes that the related model has the foreign key for the relationship.

```javascript
// UserProfile.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function user() {
        return belongsTo( "User" );
    }

}
```

## withDefault

`HasOne` relationships can be configured to return a default entity if no entity is found.  This is done by calling `withDefault` on the relationship object.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function profile() {
       return hasOne( "UserProfile" ).withDefault();
    }

}
```

Called this way will return a new unloaded entity with no data.  You can also specify any default attributes data by passing in a struct of data to `withDefault`.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function profile() {
       return hasOne( "UserProfile" ).withDefault( {
           "showHints": true
       } );
    }

}
```

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
      <td style="text-align:left">foreignKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"><code>entityName() &amp; keyNames()</code>
      </td>
      <td style="text-align:left">The foreign key on the parent entity.</td>
    </tr>
    <tr>
      <td style="text-align:left">localKey</td>
      <td style="text-align:left">String | [String]</td>
      <td style="text-align:left"><code>false</code>
      </td>
      <td style="text-align:left"><code>keyNames()</code>
      </td>
      <td style="text-align:left">The local primary key on the parent entity.</td>
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

Returns a HasOne relationship between this entity and the entity defined by `relationName`.

## Visualizer

{% embed url="https://codesandbox.io/embed/quick-has-one-visualizer-ldiyz?autoresize=1&fontsize=14&hidenavigation=1&theme=light&view=preview" %}

