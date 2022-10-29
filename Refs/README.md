# Populate

**Few important notes:**

* *Models* are defined using *Schemas*
* Instance of a model is called *document*

You can use **populate()** method to refer to the other documents.

The below examples shows **one to many relation** between *authors* and  *books* :

```ts
import mongoose from 'mongoose';

const authorSchema = new mongoose.Schema({
  name: { type: String, required: true },
  books: [{
    type: mongoose.Schema.Types.ObjectId, ref: 'Book'
  }]
})

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  price: { type: Number, required: true }, 
  author: { 
    type: mongoose.Schema.Types.ObjectId, 
    ref: 'Author'
  }
})

const Author = mongoose.model('Author', authorSchema);
const Book = mongoose.model('Book', bookSchema);
```

Here, the ref option tells Mongoose which model to use during population.

#### Note:

`ObjectId, String, Number, Buffer` all are valid types for refs. However, you should use ObjectUd unless you want to use it for advanced queries.

## Populating

```ts
const book = await Book.findOne({ name: 'Harry Potter' }).populate('author');
console.log(book.author) // returns author object
```

Similarly,

```ts
const author = await Author.findOne({ name: 'John Doe' }); 
console.log(author.books) // returns an array of book IDs 
const authorWithBooks = await Author.findOne({ name: 'John Doe' }).populate('books');
console.log(author.books) // returns an array of book objects
```

## Field Selection

What if we only want to populate other document's specific fields?
We can do that by passing in **field name** as a second argument to the populate method.

```ts
const author = await Author.findOne({ name: 'John Doe' }); 
console.log(author.books) // returns an array of book IDs 
const authorWithBooks = await Author.findOne({ name: 'John Doe' }).populate('books', 'name')
console.log(author.books) // returns an array of book objects with their names
```

## Advance Queries

What if we want to query books by an Author "John Doe" and **only** populate  **names and no ID who's price is less than 10** ?

```ts
const author = await Author.findOne({ name: "John Doe" })
                    .populate({
                      path: 'books',
                      match: {price: { $gte: 10 }},
                      select: 'name -_id'
                    })
```

More details : [https://mongoosejs.com/docs/populate.html#populate_multiple_documents](https://mongoosejs.com/docs/populate.html#populate_multiple_documents)
