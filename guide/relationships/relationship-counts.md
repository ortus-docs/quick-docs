# Relationship Aggregates

## withCount

One common type of subselect field is the count of related entites.  For instance, you may want to load a Post or a list of Posts with the count of Comments on each Post.  You can reuse your existing relationship definitions and add this count using the `withCount` method.

Adds a count of related entities as a subselect property.  Relationships can be constrained at runtime by passing a struct where the key is the relationship name and the value is a function to constrain the query.

| Name     | Type | Required | Default | Description                                                       |
| -------- | ---- | -------- | ------- | ----------------------------------------------------------------- |
| relation | any  | `true`   |         | A single relation name or array of relation names to load counts. |

By default, you will access the returned count using the relationship name appended with `Count`, i.e. `comments` will be available under `commentsCount`.

```javascript
var post = getInstance( "Post" )
	.withCount( "comments" )
	.findOrFail( 1 );
	
post.getCommentsCount();
```

You can alias the count attribute using the `AS` syntax as follows:

```javascript
var post = getInstance( "Post" )
	.withCount( "comments AS myCommentsCount" )
	.findOrFail( 1 );
	
post.getMyCommentsCount();
```

This is especially useful as you can dynamically constrain counts at runtime using the same struct syntax as eager loading with the `with` function.

```javascript
var post = getInstance( "Post" )
	.withCount( [
	    "comments AS allCommentsCount",
	    { "comments AS pendingCommentsCount": function( q ) {
	        q.where( "approved", 0 );
	    } },
	    { "comments AS approvedCommentsCount": function( q ) {
	        q.where( "approved", 1 );
	    } }
	] )
	.findOrFail( 1 );

post.getAllCommentsCount();	
post.getPendingCommentsCount();
post.getApprovedCommentsCount();
```

{% hint style="info" %}
Note that where possible it is cleaner and more readable to create a dedicated relationship instead of using dynamic constraints.  In the above example, the `Post` entity could have `pendingComments` and `approvedComments` relationships.  Dynamic constraints are more useful when applying user-provided data to the constraints, like searching.
{% endhint %}

## withSum

Adds a sum of an attribute of related entities as a subselect property.  Relationships can be constrained at runtime by passing a struct where the key is the relationship name and the value is a function to constrain the query.

| Name            | Type | Required | Default | Description                                                                                                                 |
| --------------- | ---- | -------- | ------- | --------------------------------------------------------------------------------------------------------------------------- |
| relationMapping | any  | `true`   |         | A single relation mapping string (`relationName.attributeName`) or array of relation mappings to load as summed subselects. |

By default, you will access the returned sum using the relationship name prepended with `total`, i.e. `purchases.amount` will be available under `totalPurchases`.

```javascript
var user = getInstance( "User" )
	.withSum( "purchases.amount" )
	.findOrFail( 1 );
	
user.getTotalPurchases();
```

You can alias the count attribute using the `AS` syntax as follows:

```javascript
var user = getInstance( "User" )
	.withSum( "purchases.amount AS totalPurchaseAmount" )
	.findOrFail( 1 );
	
user.getTotalPurchaseAmount();
```

This is especially useful as you can dynamically constrain counts at runtime using the same struct syntax as eager loading with the `with` function.

```javascript
var user = getInstance( "User" )
	.withCount( [
	    "purchases.amount",
	    { "purchases.amount AS totalSingleQuantityPurchases": function( q ) {
	        q.where( "quantity", 1 );
	    } },
	    { "purchases.amount AS totalMultipleQuantityPurchases": function( q ) {
	        q.where( "quantity", ">", 1 );
	    } }
	] )
	.findOrFail( 1 );

post.getTotalPurchases();	
post.getTotalSingleQuantityPurchases();
post.getTotalMultipleQuantityPurchases();
```
