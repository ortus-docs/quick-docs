# Interception Points

Quick allows you to hook in to multiple points in the entity lifecycle.

## quickPreLoad

Fired before attempting to load an entity from the database.

> This method is only called for `find` actions.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| id | The id of the entity attempting to be loaded |
| metadata | The metadata of the entity |

## quickPostLoad

Fired after loading an entity from the database.

> This method is only called for `find` actions.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity loaded |

## quickPreSave

Fired before saving an entity to the database.

> This method is called for both insert and update actions.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be saved |

## quickPostSave

Fired after saving an entity to the database.

> This method is called for both insert and update actions.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was saved |

## quickPreInsert

Fired before inserting an entity into the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be inserted |

## quickPostInsert

Fired after inserting an entity into the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was inserted |

## quickPreUpdate

Fired before updating an entity in the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be updated |

## quickPostUpdate

Fired after updating an entity in the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was updated |

## quickPreDelete

Fired before deleting a entity from the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity to be deleted |

## quickPostDelete

Fired after deleting a entity from the database.

`interceptData` structure

| Key | Description |
| :--- | :--- |
| entity | The entity that was deleted |

