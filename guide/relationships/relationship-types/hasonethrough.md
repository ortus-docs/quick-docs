# hasOneThrough

## Usage

A `hasOneThrough` relationship is either a `many-to-one` or a `one-to-one` relationship. It is used when you want to access a related entity through one or more intermediate entities. For instance, a `Team` may have one latest `Post` through a `User`. 

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" {

    function latestPost() {
        return hasOneThrough( [ "members", "posts" ] )
            .orderByDesc( "publishedDate" );
    }
    
    function members() {
        return hasMany( "User" );
    }

}
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function posts() {
        return hasMany( "Post" );
    }
    
    function team() {
        return belongsTo( "Team" );
    }

}
```

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function author() {
        return belongsTo( "Post" );
    }

}
```

The only value needed for `hasOneThrough` is an array of relationship function names to walk through to get to the related entity.  The first relationship function name in the array must exist on the current entity.  Each subsequent function name must exist on the related entity of the previous relationship result.  For our previous example, `members` must be a relationship function on `Team`.  `posts` must then be a relationship function on the related entity resulting from calling `Team.members()`.  This returns a `hasMany` relationship where the related entity is `User`.  So, `User` must have a `posts` relationship function.  That is the end of the relationship function names array, so the related entity resulting from calling `User.posts()` is our final entity which is `Post`.

```text
hasOneThrough( [ "members", "posts" ] );

+----------------+---------------------------+----------------+
| Current Entity | Relationship Method Names | Related Entity |
+================+===========================+================+
| Team           | members                   | User           |
+----------------+---------------------------+----------------+
| User           | posts                     | Post           |
+----------------+---------------------------+----------------+
```

This approach can scale to as many related entities as you need.  For instance, let's expand the previous example to include an Office that houses many Teams.

```javascript
// Office.cfc
component extends="quick.models.BaseEntity" {

    function latestPost() {
        return hasOneThrough( [ "teams", "members", "posts" ] )
            .orderByDesc( "publishedDate" );
    }
    
    function teams() {
        return hasMany( "Team" );
    }

}
```

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" {

    function members() {
        return hasMany( "User" );
    }
    
    function office() {
        return belongsTo( "Office" );
    }

}
```

```javascript
// User.cfc
component extends="quick.models.BaseEntity" {

    function team() {
        return belongsTo( "Team" );
    }
    
    function posts() {
        return hasMany( "Post" );
    }

}
```

```javascript
// Post.cfc
component extends="quick.models.BaseEntity" {

    function author() {
        return belongsTo( "User" );
    }

}
```

```text
hasOneThrough( [ "teams", "members", "posts" ] )

+----------------+---------------------------+----------------+
| Current Entity | Relationship Method Names | Related Entity |
+================+===========================+================+
| Office         | teams                     | Team           |
+----------------+---------------------------+----------------+
| Team           | members                   | User           |
+----------------+---------------------------+----------------+
| User           | posts                     | Post           |
+----------------+---------------------------+----------------+
```

## withDefault

`HasOneThrough` relationships can be configured to return a default entity if no entity is found.  This is done by calling `withDefault` on the relationship object.

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" {

    function latestPost() {
        return hasOneThrough( [ "members", "posts" ] )
            .orderByDesc( "publishedDate" )
            .withDefault();
    }
    
    function members() {
        return hasMany( "User" );
    }

}
```

Called this way will return a new unloaded entity with no data.  You can also specify any default attributes data by passing in a struct of data to `withDefault`.

```javascript
// Team.cfc
component extends="quick.models.BaseEntity" {

    function latestPost() {
        return hasOneThrough( [ "members", "posts" ] )
            .orderByDesc( "publishedDate" )
            .withDefault( {
                "title": "Your next great post!"
            } );
    }
    
    function members() {
        return hasMany( "User" );
    }

}
```

## Signature

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| relationships | array | `true` |  | An array of relationship function names. The relationships are resolved from left to right.  Each relationship will be resolved from the previously resolved relationship, starting with the current entity. |
| relationMethodName | string | false | Current Method Name | The method name called to retrieve this relationship.  Uses a stack backtrace to determine by default. |

