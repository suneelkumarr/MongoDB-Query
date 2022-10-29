
# Indexing

```js
{
        "_id" : ObjectId("603a87095854104ef6c86425"),
        "gender" : "female",
        "name" : {
                "title" : "ms",
                "first" : "kelya",
                "last" : "philippe"
        },
        "location" : {
                "street" : "3688 quai chauveau",
                "city" : "avignon",
                "state" : "aisne",
                "postcode" : 47002,
                "coordinates" : {
                        "latitude" : "2.4082",
                        "longitude" : "153.9632"
                },
                "timezone" : {
                        "offset" : "+4:00",
                        "description" : "Abu Dhabi, Muscat, Baku, Tbilisi"
                }
        },
        "dob" : {
                "date" : "1950-08-05T15:04:26Z",
                "age" : 68
        }
}
```

 **To check the speed of your search query analysis** :

```js
db.contacts.explain('executionStats').find({ 'dob.age': { $gt: 60 } });
```

## Creating index

Let us say we need to create an index on **age**

```js
/*
 *  1    ==    ascending ordered index
 * -1    ==    descending ordered index
 *  The speed doesn't depend on the sort much
 *  because mongo can find the document from either direction
 */
db.contacts.createIndex({ 'dob.age': 1 });
```

### EXPLAINATION :

Index scans (index stage) does not return the documents. They return the pointers to the documents.
Later on, the *fetch stage* reach out to the actual document using that pointer.

### CAVIAT

Let us say we want all the users with the ages greater than 10:

```js
// assuming indexing is still there
db.contacts.find({ 'dob.age': { $gt: 10 } });
```

 *This execution will actually be slower than the one WITHOUT INDEXING* .

*WHY IS THAT?*

This is because `age > 10` covers 90% of the
 documents inside of the database.
So our database had to cover 90% of the indexes and returns all the
pointers = pointing to their respective databases. And further, it took
time to fetch those documents for us, so it actually was slow.

The point is, you should not be using indexes for the queries which return a gigantic number of documents.
*Rather, use indexes for fields which are usually unique and return less amount of documents*

## Deleting index

To delete the index:

```js
db.contacts.dropIndex({ 'dob.age': 1 });
```

*_id field has a default indexed*

## Getting indexes

To find all the existing indexes on a collection:

```js
db.contacts.getIndexes();
```

## Creating Unique Index

Mongo has _id as default index since it is unique. Let us say we have collection of users who's email IDs are ALWAYS unique **AND** you want to query user using email field  **frequently** , then you can create a new unique index like:

```js
db.users.createIndex({ email: 1 }, { unique: true });
```

## Creating COMPOUND INDEXES

This is used to create indexes using two fields in your collection:

```js
// can be used together or from left -> right (see examples)
db.users.createIndex({ 'dob.age': 1, gender: 1 });
```

This would create an index field something like: `33 male`.

### Examples

Query1 (good query for indexes):

```js
// order does not matter (can be different from that of index)
db.users.find({ 'dob.age': 35, gender: 'male' });
```

Query2 (also fine `left to right`):

```js
// this will also use same index
db.users.find({ 'dob.age': 35 });
```

Query3 (WRONG this won't use index)

```js
// query is right but it won't make use of index
// if you move left to right, you have to include the left ones
db.users.find({ gender: 'male' });
```

## Sorting using Indexes

In case you need large amount of data to be sorted using a
 specific field in the document on a regular basis, you should use
indexing for that partifcular field because it helps the response time
by ALOT as the db wouldn't have to sort the returned data for you. This
is because indexes are already sorted.

## PARTIAL FILTERS

Let us say you create indexes on age BUT you use the age query only for the gener male everytime in the users collection.

So, there is no point of making index on the whole collection. This is what you would do:

```js
db.users.createIndex({ 'age.dob': 1 }, { partialFilterExpression: { gender: 'male' } });
```

You could also do something like :

```js
db.users.createIndex(
  { 'age.dob': 1 },
  { partialFilterExpression: { 'hobbies.frequency': { $gt: 6 } } } // where
);
```

**NOTE:** : *to use this index, you will
have to include exact indexes while querying. This won't work like
compound indexes(left -> right). This is because, if mongo does that,
 since the indexes are partial, you might end up skipping returned data.
 HMMMMM....? Here, what if the the qeury satisfies the documents on
which there is no indexing?*

Further explaination:

Let us consider the partial indexing:

```js
db.users.createIndex({ 'dob.age': 1 }, { partialFilterExpression: { gender: 'male' } });
```

This create indexes ONLY FOR MALES:

Now if you query for people older than 60:

```js
db.users.find({ 'dob.age': 60 });
```

This^^ will NOT run IndexScan, BECAUSE what if we have
users who are women and are > 60 age? And if we were to use index for
 scanning, it would have skipped those documents.

*Therefore, mongo will run a collection scan*

**The best way to use this index would be:**

```js
db.users.find({ 'db.age': 60, gender: 'male' });
```

## TEXT INDEXES (usually used for search functionality)

Dataset:

```js
{
        title: "Book1",
        description: "This is an awesome book and is a must buy"
}
```

Command:

```js
// use the word 'text' instead of 1/-1
db.products.createIndex({ description: 'text' });
```

The collection must have ONLY ONE TEXT INDEX since it is very expensive.

This will create an ARRAY of index with the words `awesome`, `book`, `must`, `buy` etc and ignore the rest of the generic words for us.

**Search Query**

```js
db.products.find({ $text: { $search: 'awesome' } });
```

```js
// this will return all the documents with the words 'awesome' and 'book'
db.products.find({ $text: { $search: 'awesome book' } });
```

Let us say we want documents with specificly `awesome book` as a single word:

```js
db.products.find({ $text: { $search: '"awesome book"' } });
```

This will only return if the description has "awesome book" as a phrase.

### SORT TEXT INDEXES RESULTS BASED ON EXACT MATCH (SCORE)

Dataset:

```js
[
  {
    title: 'Red T-Shirt',
    description: 'This T-shirt is read and is awesome',
  },
  {
    title: 'A Book',
    description: 'This is an awesome book about a young artist',
  },
];
```

Let us search for "awesome t-shirt" ( *which will return both the docs as they both have either of the words* ).
But I want it to sort it based on the  **most accurate match** .

*SOLUTION*

```js
db.products.find(
  { $text: { $search: ' awesome book ' } },
  { score: { $meta: 'textScore' } } // this sorts it
);
```

We can further change the order of sorting:

```js
db.products
  .find({ $text: { $search: 'awesome book' } }, { score: { $meta: 'textScore' } })
  .sort({ score: -1 }); // here
```
