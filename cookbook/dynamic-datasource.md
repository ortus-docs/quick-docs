---
description: Contributed by Wil de Bruin
---

# Dynamic Datasource

{% hint style="info" %}
Read about a real world usage of this pattern [on Wil's blog.](https://shiftinsert.nl/dynamic-datasources-part-2-quick/)
{% endhint %}

## Problem

Often you will configure Quick with a default datasource, but in some cases you might need a dynamic datasource. For example, your killer app is multi-tenant, each customer has it's own database but the codebase is exactly the same for each customer. In this case you have two options:

* Deploy the same app over and over again for each customer, which is only feasible for a small customer count.
*   or create some central authentication mechanisms, and after authentication each user gets his own

    * userId
    * customerId
    * customerDatasource

    In this scenario you know from your user authentication info which datasource you should use for each customer. The problem: this datasource name is dynamic, while Quick knows about default datasources or [fixed named datasources](https://quick.ortusbooks.com/guide-1/getting-started/defining-an-entity#multiple-datasource-support). So how can we make this datasource dynamic?

## Solution

Actually there are multiple solutions available - you just have to pick one which suits you best. After login you know which datasource you need. Since Quick needs to know where to get the value of your datasource you need to store it somewhere. We will store it in the private request collection (`prc`), but other options like session storage or user-specific caching might be preferable in your use case. So let's assume we have a variable called `mainDatasource`  in our private request collection which has the datasource name we should use.  We should be able to change the default datasource in every quick object just before it sends a query to the database.&#x20;

### Option 1: instanceReady()

Quick fires an `instanceReady` event internally and as an interceptor.  We will hook in to this lifecycle event by creating a base component for our entities.

```javascript
component displayname="DynamicQuickBase" extends="quick.models.BaseEntity" {

	function instanceReady() {
		var thisDataSource = variables._wirebox.getInstance( "coldbox:requestContext" ).getPrivateValue( "mainDatasource","" );
		if ( len( thisDataSource) ){
			variables._queryoptions["datasource"] = thisDataSource;
			variables._builder.setDefaultOptions( variables._queryoptions );
		}
	}
}
```

In this case, I use the `instanceReady()` lifecycle method which fires **AFTER** an object is created, but before data is loaded from the database. We get the datasource from the private request collection, and only take action if it is set. If there is a value, we change the `_queryoptions` on the builder object. Unfortunately this is not enough, because the  `_queryoptions` will not be read again after `instanceReady`fires, so we explicitly have to set the default options on the \_builder object again. Once that's done, your Quick entity is ready to query using your dynamic datasource. So every Quick entity which inherits from our custom base component will use dynamic datasources

### Option 2: overriding the newQuery() method

The `newQuery` method reads the default `_queryOptions`, and prepares a query object. We create a base component again and this time override the `newQuery` method.

```javascript
component displayname="DynamicQuickBase" extends="quick.models.BaseEntity" {

	/**
	 * Configures a new query builder and returns it.
	 * We modify it to get a dynamic datasource
	 *
	 * @return  quick.models.QuickBuilder
	 */
	public any function newQuery() {
    var thisDataSource = variables._wirebox.getInstance( "coldbox:requestContext" ).getPrivateValue( "customerDatasource","" );
		if ( len( thisDataSource) ){
			variables._queryoptions["datasource"] = thisDataSource;
		}
		return super.newQuery();
	}
}
```

In this scenario we read the dynamic datasource value from the `prc` again, and modify the `_queryoptions` to use that datasource. Once this is ready we can call the `newQuery()` method of the parent object to finish setting up. Again, if your Quick entity inherits from this base, your quick object will now be using dynamic datasources.

### Option 3: Intercepting qb execution

The third option might look simpler, but offers you less control. Quick uses qb to query the database, and just before it does this it announces a `preQBExecute` interception point. By creating and registering an interceptor, you can change your datasource just in time. The interceptor can look like this:

{% code title="QBDynamicDataSourceInterceptor.cfc" %}
```javascript
component {

    function preQBExecute( event, data, buffer, rc, prc ){
        var thisDataSource = event.getPrivateValue( "mainDatasource", "" );
        if ( len( thisDataSource ) && !data.options.keyExists( "dataSource" ) ) {
            data.options["dataSource"] = thisDataSource;
        }
    }

}	
```
{% endcode %}

In this case you don't need your own base component. Be aware that this interceptor changes your datasource for ALL qb requests, no matter if you use custom base component or even if it is a regular qb query. So if qb already had some default datasource that will also be changed. We added an extra check, so this interceptor is not touching your qb query if a datasource was already explicitly defined. You can register your interceptor in the coldbox config like this

```javascript
interceptors = [
    { class: "interceptors.QbDynamicDatasourceInterceptor" }
			//.. more
];
```

Option 2 is the most specific option you can use, and makes most sense if you see what happens in the `newQuery` method of `quick.models.BaseEntity`. Option 3 is less specific and might change your datasource in unwanted places. But if you know what you are doing it can be a very powerful way to add this dynamic datasource capability to Quick.
