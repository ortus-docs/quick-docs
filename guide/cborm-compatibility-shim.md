# CBORM Compatibility Shim

To assist you in migrating from CBORM, Quick ships with a small compatibility shim. To use it, have your entity extend `quick.models.CBORMCompatEntity`. This will map common CBORM methods to their Quick counterparts as well as provide a partial CriteriaBuilder shim. The compatibility shim does not cover differences in properties or relationships.

## Entity / Service Methods

* list
* countWhere
* deleteById
* deleteWhere
* exists
* findAllWhere
* findWhere
* get
* getAll
* new
* populate
* save
* saveAll
* newCriteria

## Criteria Builder Methods

* getSQL
* between
* eqProperty
* isEQ
* isGT
* gtProperty
* isGE
* geProperty
* idEQ
* like
* ilike
* isIn
* isNull
* isNotNull
* isLT
* ltProperty
* neProperty
* isLE
* leProperty
* maxResults
* firstResult
* order
* list
* get
* count
* onMissingMethod
