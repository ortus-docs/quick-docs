# Defining Child Classes

By default, Quick supports basic component-level inheritance of entities, meaning that a child component inherits the properties ( and ability to overload ) its parent. A common, object-oriented relational database pattern, however is to provide additional definition on parent tables ( and classes ) within child tables which contain a foreign key.  

Quick supports two types of child classes:  Discriminated and Subclassed child entities. In both cases, loading any child class will also deliver the data of its parent class.

## SubClass Entities

Let's say, for example, that I have a `Media` entity, which is used to catalog and organize all media items loaded in to my application.  

```javascript
component 
    table="media" 
    extends="quick.models.BaseEntity" 
    accessors="true"
{

    property name="id";
    property name="uploadFileName";
    property name="fileLocation";
    property name="fileSizeBytes";

}
```

My `Media` entity contains all of the properties which are common to every media item uploaded in to my application. Let's say, however, that I need to have specific attributes that are available on only media for my `Book` entity ( e.g. whether the image is the cover photo, for example ).  I can create a child class of `BookMedia` which extends my `Media` entity.  When loaded, all of the properties of `Media` will be loaded along with the custom attributes which apply to my `BookMedia` object:

```javascript
component
    extends="Media"
    table="book_media"
    joinColumn="FK_media"
    accessors="true"
{
    property name="displayOrder";
    property name="designation";

    function approvalStatus(){
        return belongsTo( "Book", "FK_book" );
    }

}
```

Note the additional component attribute `joincolumn`.  The presence of this attribute on a child class signifies that it is a child entity of the parent and that the parent's properties should be loaded whenever the `BookMedia` entity is loaded. In addition, the primary key of the entity is that of the parent. Child entities can be retrieved by queries specific to their own properties: 

```javascript
var coverPhotos = getInstance( "BookMedia" )
                    .where( "designation", "cover" )
                    .orderBy( "displayOrder", "ASC" );

```

Or properties on the parent class can be used as first-class properties within the query:

```javascript
var smallCoverPhotos = getInstance( "BookMedia" )
                    .where( "designation", "cover" )
                    .where( "fileSizeBytes", "<", 40000 )
                    .orderBy( "displayOrder", "ASC" )
                    .orderBy( "uploadFileName", "ASC" );
```

Child entities can be retreived, individually, using the value of the `joinColumn`, which should be a foreign key to the parent identifier column:

```
var myBookMediaItem = getInstance( "BookMedia" ).get( myId );
```

Now my `Book` entity can use its extended media class to retreive media items which are specific to its own purpose:

```javascript

function media(){
    return hasMany( "BookMedia", "FK_book" ).orderBy( "displayOrder", "ASC" );
}

```


## Discriminated Entities

A discriminated child class functions, basically, in the same way as a subclassed entity, with one exception:  The parent entity is aware of the discriminated child, due to a `discriminatorValue` attribute and will return that specific class when a retreival is performed through the parent Entity. Let's take our `BookMedia` class, again, but this time defining it as a discriminated entity.  

The first step is to add the `discriminatorColumn` attribute to the `Media` entity:

```javascript
component 
    table="media" 
    extends="quick.models.BaseEntity" 
    accessors="true"
    discriminatorColumn="type"
{

    property name="id";
    property name="uploadFileName";
    property name="fileLocation";
    property name="fileSizeBytes";

}
```

Then we set a `discriminatorValue` property on the child class, the value of which is stored in the parent entity table:

```javascript
component
    extends="Media"
    table="book_media"
    joinColumn="FK_media"
    discriminatorValue="book"
    accessors="true"
{
    property name="displayOrder";
    property name="designation";

    function approvalStatus(){
        return belongsTo( "Book", "FK_book" );
    }

}
```

Once this is defined, any new `BookMedia` entity will be saved with a `type` value of `book` in the `media` table.  As such, the following query will result in only entities of `BookMedia` being returned:

```javascript
var bookMedia = getInstance( "Media" ).where( "type", "book" ).get();
```

If our `Media` table contains a combination of non-book and book media, then the collection returned when querying all records will contain a mix of `BookMedia` and `Media` entities.  

Loading a collection of `BookMedia` entities, however, will always return a collection of `BookMedia` entities, because the `type` column value on the `media` must be equal to `book`.


## Summary

Discriminated and child class entities, allow for a more Object oriented approach to entity-specific relationships by allowing you to eliminate pivot/join tables and extend the attributes of the base class.
