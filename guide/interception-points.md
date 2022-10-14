# Interception Points

Quick allows you to hook in to multiple points in the entity lifecycle. If the event is on the component, you do not need to prefix it with `quick`. If you are listening to an interception point, include `quick` at the beginning.

{% hint style="warning" %}
If you create your own Interceptors, they will not fire if you define them in your Main application. `quick` will be loaded AFTER your interceptors, so the `quick` interception points will **not** be registered with your interceptor. This can be solved by moving your interceptors to a module with a dependency on `quick`, of by also registering the `quick` custom interception points in your main coldbox configuration.
{% endhint %}

## quickInstanceReady

Fired after dependency injection has been performed on the entity and the metadata has been inspected.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity loaded |
| entityName | The name of the entity |


## quickPreLoad

Fired before attempting to load an entity from the database.

{% hint style="warning" %}
This method is only called for `find` actions.
{% endhint %}

`eventData` structure

| Key | Description |
| :--- | :--- |
| id | The id of the entity attempting to be loaded |
| metadata | The metadata of the entity |
| entityName | The name of the entity |

## quickPostLoad

Fired after loading an entity from the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity loaded |
| entityName | The name of the entity |

## quickPreSave

Fired before saving an entity to the database.

> This method is called for both insert and update actions.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be saved |
| entityName | The name of the entity |

## quickPostSave

Fired after saving an entity to the database.

> This method is called for both insert and update actions.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was saved |
| entityName | The name of the entity |

## quickPreInsert

Fired before inserting an entity into the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be inserted |
| entityName | The name of the entity |

## quickPostInsert

Fired after inserting an entity into the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was inserted |
| entityName | The name of the entity |

## quickPreUpdate

Fired before updating an entity in the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be updated |
| entityName | The name of the entity |

## quickPostUpdate

Fired after updating an entity in the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was updated |
| entityName | The name of the entity |

## quickPreDelete

Fired before deleting a entity from the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be deleted |
| entityName | The name of the entity |

## quickPostDelete

Fired after deleting a entity from the database.

`eventData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was deleted |
| entityName | The name of the entity |

