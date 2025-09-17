# hasManyDeep

## Usage

A `hasManyDeep` relationship is either a `one-to-many` or a `many-to-many` relationship. It is used when you want to access related entities through one or more intermediate entities.  For example, you may want to retrieve all the blog posts for a `Team`:

```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function posts() {
        return hasManyDeep(
            relationName = "Post",
            through = [ "User" ],
            foreignKeys = [
                "teamId", // the key on User that refers to Team
                "authorId" // the key on Post that refers to User
            ],
            localKeys = [
                "id", // the key on Team that identifies Team
                "id" // the key on User that identifies User
            ]
        );
    }

}
```

This would generate the following SQL:

```sql
SELECT *
FROM `posts`
INNER JOIN `users`
    ON `posts`.`authorId` = `users`.`id`
WHERE `users`.`teamId` = ? // team.getId()
```

You can generate this same relationship using a Builder syntax:

```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function posts() {
        return newHasManyDeepBuilder()
            .throughEntity( "User", "teamId", "id" )
            .toRelated( "Post", "authorId", "id" );
    }

}
```

`HasManyDeep` relationships can also traverse pivot tables. For instance, a `User` may have multiple `Permissions` via a `Role` entity.

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function permissions() {
        return hasManyDeep(
            relationName = "Role",
            through = [ "roles_users", "Role", "permissions_roles" ],
            foreignKeys = [ "userId", "id", "roleId", "id" ],
            localKeys = [ "id", "roleId", "id", "permissionId" ]
        );
    }

}
```

{% hint style="info" %}
If a mapping in the `through` array is found as a WireBox mapping, it will be treated as a Pivot Table.
{% endhint %}

{% hint style="info" %}
If you are traversing a `polymorhpic` relationship, pass an array as the key type where the first key points to the polymorphic type column and the second key points to the identifying value.
{% endhint %}

## Constraining Relationships

There are two options for constraining a `HasManyDeep` relationship.

The first option is by using table aliases.

{% tabs %}
{% tab title="Traditional Syntax" %}
```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function activePosts() {
        return hasManyDeep(
            relationName = "Post",
            through = [ "User AS u" ],
            foreignKeys = [
                "teamId", // the key on User that refers to Team
                "authorId" // the key on Post that refers to User
            ],
            localKeys = [
                "id", // the key on Team that identifies Team
                "id" // the key on User that identifies User
            ]
        ).where( "u.active", 1 );
    }

}
```
{% endtab %}

{% tab title="Builder Syntax" %}
```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function activePosts() {
        return newHasManyDeepBuilder()
            .throughEntity( "User AS u", "teamId", "id" )
            .toRelated( "Post", "authorId", "id" )
            .where( "u.active", 1 );
    }

}
```
{% endtab %}
{% endtabs %}

This produces the following SQL:

```sql
SELECT *
FROM `posts`
INNER JOIN `users` AS `u`
    ON `posts`.`authorId` = `users`.`id`
WHERE `u`.`teamId` = ? // team.getId()
AND `u`.`active` = ? // 1
```

If you want to use scopes or avoid table aliases, you can use callback functions to constrain the relationship:

{% tabs %}
{% tab title="Traditional Syntax" %}
```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function activePosts() {
        return hasManyDeep(
            relationName = "Post",
            through = [ () => newEntity( "User ).active() ],
            foreignKeys = [
                "teamId", // the key on User that refers to Team
                "authorId" // the key on Post that refers to User
            ],
            localKeys = [
                "id", // the key on Team that identifies Team
                "id" // the key on User that identifies User
            ]
        );
    }

}
```
{% endtab %}

{% tab title="Builder Syntax" %}
```cfscript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function activePosts() {
        return newHasManyDeepBuilder()
            .throughEntity(
                entityName = "User",
                foreignKey = "teamId",
                localKey = "id",
                callback = ( user ) => user.active()
            )
            .toRelated( "Post", "authorId", "id" )
    }

}
```
{% endtab %}
{% endtabs %}

## Signature

<table data-header-hidden><thead><tr><th>Name</th><th>Type</th><th data-type="checkbox">Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>relationName</td><td><code>String</code> | <code>Function</code> | <code>QuickBuilder</code> | <code>IRelationship</code></td><td>true</td><td></td><td>A mapping name to the final related entity, a function that produces a <code>QuickBuilder</code> or <code>Relationship</code> instance, or a <code>QuickBuilder</code> or <code>Relationship</code> instance.</td></tr><tr><td>through</td><td>Array&#x3C;<code>String</code> | <code>Function</code> | <code>QuickBuilder</code> | <code>IRelationship</code>></td><td>true</td><td></td><td><p>The entities to traverse through from the parent (current) entity to get to the <code>relatedEntity</code>.  </p><p></p><p>Each item in the array can be either a mapping name to the final related entity, a function that produces a <code>QuickBuilder</code> or <code>Relationship</code> instance, or a <code>QuickBuilder</code> or <code>Relationship</code> instance.</p></td></tr><tr><td>foreignKeys</td><td><code>Array&#x3C;String></code></td><td>true</td><td></td><td>An array of foreign keys traversing to the <code>relatedEntity</code>.<br><br>The length of this array should be one more than the length of <code>through</code>.<br><br>The first key of this array would be the column on the first <code>through</code> entity that refers back to the parent (current) entity, and so forth. </td></tr><tr><td>localKeys</td><td><code>Array&#x3C;String></code></td><td>true</td><td></td><td>An array of local keys traversing to the <code>relatedEntity</code>.<br><br>The length of this array should be one more than the length of <code>through</code>.<br><br>The first key of this array would be the column on the parent (current) entity that identifies it, and so forth.</td></tr><tr><td>nested</td><td><code>boolean</code></td><td>false</td><td><code>false</code></td><td>Signals the relationship that it is currently being resolved as part of another <code>hasManyDeep</code> relationship.  This is handled by the framework when using a <code>hasManyThrough</code> relationship.</td></tr><tr><td>relationMethodName</td><td><code>String</code></td><td>false</td><td>Current Method Name</td><td>The method name called to retrieve this relationship.  Uses a stack backtrace to determine by default.</td></tr></tbody></table>

