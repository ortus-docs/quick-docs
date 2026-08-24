# Interception Points

Quick allows you to hook in to multiple points in the entity lifecycle. If the event is on the component, you do not need to prefix it with `quick`. If you are listening to an interception point, include `quick` at the beginning.

{% hint style="warning" %}
If you create your own Interceptors, they will not fire if you define them in your Main application. `quick` will be loaded AFTER your interceptors, so the `quick` interception points will **not** be registered with your interceptor. This can be solved by moving your interceptors to a module with a dependency on `quick`, or by also registering the `quick` custom interception points in your main ColdBox configuration.
{% endhint %}

## Custom entity lifecycle events

An entity can map a Quick lifecycle event to one or more application-specific interception points using `_dispatchesEvents`:

```javascript
component extends="quick.models.BaseEntity" {

    variables._dispatchesEvents = {
        "postInsert" : "onOrderCreated",
        "postSave"   : [ "onOrderSaved", "onAggregateChanged" ]
    };

}
```

Quick announces each configured custom point when it fires the corresponding lifecycle event. The custom point receives the same `eventData` as the standard Quick interception point.

{% hint style="warning" %}
`_dispatchesEvents` does not register the custom interception points with ColdBox. Every configured point must also be registered in your ColdBox application or module before an interceptor can listen to it.
{% endhint %}

For example, register the names in `config/Coldbox.cfc`:

```javascript
interceptorSettings = {
    customInterceptionPoints : [
        "onOrderCreated",
        "onOrderSaved",
        "onAggregateChanged"
    ]
};
```

Alternatively, an interceptor method can register its own custom point using ColdBox's `interceptionPoint` annotation:

```javascript
/**
 * @interceptionPoint
 */
function onOrderCreated( event, interceptData ) {
    // ...
}
```

The annotation registers the custom point with ColdBox; the entity's `_dispatchesEvents` map still defines which Quick lifecycle event announces it.

## quickInstanceReady

Fired after dependency injection has been performed on the entity and the metadata has been inspected.

`interceptData` structure

| Key    | Description       |
| ------ | ----------------- |
| entity | The entity loaded |

## quickPreLoad

Fired before attempting to load an entity from the database.

{% hint style="warning" %}
This method is only called for `find` actions.
{% endhint %}

`interceptData` structure

| Key      | Description                                  |
| -------- | -------------------------------------------- |
| id       | The id of the entity attempting to be loaded |
| metadata | The metadata of the entity                   |

## quickPostLoad

Fired after loading an entity from the database.

`interceptData` structure

| Key    | Description       |
| ------ | ----------------- |
| entity | The entity loaded |

## quickPreSave

Fired before saving an entity to the database.

> This method is called for both insert and update actions.

`interceptData` structure

| Key    | Description            |
| ------ | ---------------------- |
| entity | The entity to be saved |

## quickPostSave

Fired after saving an entity to the database.

> This method is called for both insert and update actions.

`interceptData` structure

| Key    | Description               |
| ------ | ------------------------- |
| entity | The entity that was saved |

## quickPreInsert

Fired before inserting an entity into the database.

`interceptData` structure

| Key        | Description                              |
| ---------- | ---------------------------------------- |
| entity     | The entity to be inserted                |
| builder    | The builder instance doing the inserting |
| attributes | The attributes data for the entity       |

## quickPostInsert

Fired after inserting an entity into the database.

`interceptData` structure

| Key    | Description                  |
| ------ | ---------------------------- |
| entity | The entity that was inserted |

## quickPreUpdate

Fired before updating an entity in the database.

`interceptData` structure

| Key    | Description              |
| ------ | ------------------------ |
| entity | The entity to be updated |

## quickPostUpdate

Fired after updating an entity in the database.

`interceptData` structure

| Key                | Description                                                    |
| ------------------ | -------------------------------------------------------------- |
| entity             | The entity that was updated                                    |
| newAttributes      | A struct of new attributes about to be updated.                |
| originalAttributes | A struct of attributes at the time the entity was last loaded. |

## quickPreDelete

Fired before deleting a entity from the database.

`interceptData` structure

| Key    | Description              |
| ------ | ------------------------ |
| entity | The entity to be deleted |

## quickPostDelete

Fired after deleting a entity from the database.

`interceptData` structure

| Key    | Description                 |
| ------ | --------------------------- |
| entity | The entity that was deleted |
