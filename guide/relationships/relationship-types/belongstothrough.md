# belongsToThrough

## Usage

A `belongsToThrough` relationship is the `one` side of a `many-to-one` relationship with an intermediate entity. It is used when you want to access a related, owning entity through one or more intermediate entities. For instance, a `Post` may belong to a `Team` via a `User`.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function team() {
        return belongsToThrough( [ "author", "team" ] );
    }

    function author() {
        return belongsTo( "Post" );
    }

}
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function posts() {
        return hasMany( "Post" );
    }

    function team() {
        return belongsTo( "Team" );
    }

}
```

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function members() {
        return hasMany( "User" );
    }

}
```

The only value needed for `belongsToThrough` is an array of relationship function names to walk through to get to the related entity. The first relationship function name in the array must exist on the current entity. Each subsequent function name must exist on the related entity of the previous relationship result. For our previous example, `author` must be a relationship function on `Post`. `team` must then be a relationship function on the related entity resulting from calling `Post.author()`. This returns a `belongsTo` relationship where the related entity is `User`. So, `User` must have a `team` relationship function. That is the end of the relationship function names array, so the related entity resulting from calling `User.team()` is our final entity which is `Team`.

```text
belongsToThrough( [ "author", "team" ] );

+----------------+---------------------------+----------------+
| Current Entity | Relationship Method Names | Related Entity |
+================+===========================+================+
| Post           | author                    | User           |
+----------------+---------------------------+----------------+
| User           | team                      | Team           |
+----------------+---------------------------+----------------+
```

This approach can scale to as many related entities as you need. For instance, let's expand the previous example to include an Office that houses many Teams.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function office() {
        return belongsToThrough( [ "author", "team", "office" ] );
    }

    function author() {
        return belongsTo( "Post" );
    }

}
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function posts() {
        return hasMany( "Post" );
    }

    function team() {
        return belongsTo( "Team" );
    }

}
```

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function members() {
        return hasMany( "User" );
    }

    function office() {
        return belongsTo( "Office" );
    }

}
```

```javascript
// Office.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function teams() {
        return hasMany( "Team" );
    }

}
```

```text
belongsToThrough( [ "author", "team", "office" ] );

+----------------+---------------------------+----------------+
| Current Entity | Relationship Method Names | Related Entity |
+================+===========================+================+
| Post           | author                    | User           |
+----------------+---------------------------+----------------+
| User           | team                      | Team           |
+----------------+---------------------------+----------------+
| Team           | office                    | Office         |
+----------------+---------------------------+----------------+
```

## withDefault

`HasOneThrough` relationships can be configured to return a default entity if no entity is found. This is done by calling `withDefault` on the relationship object.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function team() {
        return belongsToThrough( [ "author", "team" ] ).withDefault();
    }

    function author() {
        return belongsTo( "Post" );
    }

}
```

Called this way will return a new unloaded entity with no data. You can also specify any default attributes data by passing in a struct of data to `withDefault`.

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" accessors="true" {

    function team() {
        return belongsToThrough( [ "author", "team" ] ).withDefault( {
            "name": "No Team"
        } );
    }

    function author() {
        return belongsTo( "Post" );
    }

}
```

## Signature

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| relationships | array | `true` |  | An array of relationship function names. The relationships are resolved from left to right.  Each relationship will be resolved from the previously resolved relationship, starting with the current entity. |
| relationMethodName | string | false | Current Method Name | The method name called to retrieve this relationship.  Uses a stack backtrace to determine by default. |

