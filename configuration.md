# Configuration

The following are the module settings for Quick:

```cfscript
settings = {
    "defaultGrammar"        : "AutoDiscover@qb",
    "defaultQueryOptions"   : {},
    "preventDuplicateJoins" : true,
    "metadataCache"         : {
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
