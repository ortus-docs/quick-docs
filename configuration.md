# Configuration

The following are the module settings for Quick:

```cfscript
settings = {
    "defaultGrammar"             : "AutoDiscover@qb",
    "defaultQueryOptions"        : {},
    "preventDuplicateJoins"      : true,
    "preventLazyLoading"         : false,
    "automaticTimestamps"        : true,
    "refreshOnSaveFallback"      : true,
    "lazyLoadingViolationCallback" : ( entity, relationName ) => {
        throw(
            type = "QuickLazyLoadingException",
            message = "Attempted to lazy load [#relationName#] on [#entity.mappingName()#]."
        );
    },
    "metadataCache"              : {
        "name"       : "quickMeta",
        "provider"   : "coldbox.system.cache.providers.CacheBoxColdBoxProvider",
        "properties" : {
            "objectDefaultTimeout"  : 0, // no timeout
            "useLastAccessTimeouts" : false, // no last access timeout
            "maxObjects"            : 300,
            "objectStore"           : "ConcurrentStore"
        }
    }
};
```

## automaticTimestamps

When `true`, Quick automatically maintains conventional `createdDate` and `modifiedDate` attributes during entity saves and bulk mutations. This defaults to `true` in Quick 13. See [Automatic Timestamps](guide/getting-started/automatic-timestamps.md) for entity-level and query-level configuration.

## refreshOnSaveFallback

Attributes marked `refreshOnSave="true"` are populated from the database after an insert or update. Quick uses the write statement's native `RETURNING` or `OUTPUT` support when available. When the database cannot return the values directly, this setting allows one narrow follow-up query by primary key. It defaults to `true`.

The fallback can also be disabled for one save:

```javascript
user.save( refreshOnSaveFallback = false );
```
