# Ordering By Relationships

To order by a relationship field, you will use a dot-delimited syntax.

```javascript
getInstance( "Post" ).orderBy( "author.name" );
```

The last item in the dot-delimited string should be an attribute on the related entity.

Nested relationships are also supported.  Continue to chain relationships in your dot-delimited string until arriving at the desired entity.  Remember to end with the attribute to order by.

```javascript
getInstance( "Post" ).orderBy( "author.team.name" );
```

If you desire to be explicit, you can use the `orderByRelated` method, which is what is being called under the hood.

| Name | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| relationshipName | String \| \[String\] | `true` |  | A dot-delimited string of relationship names or an array of relationship names to traverse to get to the related entity to order by. |
| columnName | string | `true` |  | The column name in the final relationship to order by. |
| direction | string | `false` | `"asc"` | The direction to sort, `asc` or `desc`. |

```javascript
getInstance( "Post" ).orderByRelated( "author.team", "name" );
// or
getInstance( "Post" ).orderByRelated( [ "author", "team" ], "name" );
```

You might prefer the explicitness of this method, but it cannot handle normal orders like `orderBy`.  Use whichever method you prefer.

