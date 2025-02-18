# MongoDB Fundamentals Assignment

## Objective:
This assignment aims to apply MongoDB concepts learned throughout the week, focusing on CRUD operations, data modeling, aggregation, indexing, and other key MongoDB skills.

---

## Setup MongoDB

### 1. Verify MongoDB Installation
Open terminal and verify MongoDB installation with the following commands:
**mongod --version**

Start MongoShell

**mongosh**

### 2. Create the Library

**use library**

**db.createCollection('books')**

### 3. Insert Multiple Books
** db.books.insertMany([
  {
    title: "To Kill a Mockingbird",
    author: "Harper Lee",
    publishedYear: 1960,
    genre: "Fiction",
    ISBN: "978-0061120084"
  },
  {
    title: "1984",
    author: "George Orwell",
    publishedYear: 1949,
    genre: "Dystopian",
    ISBN: "978-0451524935"
  },
  {
    title: "The Great Gatsby",
    author: "F. Scott Fitzgerald",
    publishedYear: 1925,
    genre: "Classic",
    ISBN: "978-0743273565"
  },
  {
    title: "Moby-Dick",
    author: "Herman Melville",
    publishedYear: 1851,
    genre: "Adventure",
    ISBN: "978-1503280786"
  },
  {
    title: "Pride and Prejudice",
    author: "Jane Austen",
    publishedYear: 2013,
    genre: "Romance",
    ISBN: "978-1503290563"
  }
]) **

Retrieving Data
### 4. Query Books
   Retrieve all books:

         db.books.find().pretty()

  Query books by author (e.g., Harper Lee):

      db.books.find({ author: "Harper Lee" })

Find books published after the year 2000:

    db.books.find({ publishedYear: { $gt: 2000 } })

Updating Data
### 5. Update Documents

Update the published year of a book (e.g., "The Great Gatsby"):

      db.books.updateOne(
        { title: "The Great Gatsby" },
        { $set: { publishedYear: 1926 } }
      )

Add a new field rating to all books:
      
      db.books.updateMany(
        {},
        { $set: { rating: 0 } }
      )

Deleting Data
### 6. Remove Documents

Delete a book by ISBN (e.g., "Pride and Prejudice"):

    db.books.deleteOne({ ISBN: "978-1503290563" })

Remove all books of a particular genre (e.g., "Adventure"):

    db.books.deleteMany({ genre: "Adventure" })

### 7 Collection Design
## i. Users Collection

The users collection stores details about each user, including profile information, orders, and shopping cart.

    db.users.insertOne({
      username: "john_doe",
      email: "john.doe@example.com",
      passwordHash: "hashed_password",
      fullName: "John Doe",
      address: {
        street: "123 Main St",
        city: "Some City",
        zipCode: "12345"
      },
      phone: "123-456-7890",
      orders: [ObjectId("order_id_1"), ObjectId("order_id_2")],
      cart: [
        { productId: ObjectId("product_id_1"), quantity: 2 },
        { productId: ObjectId("product_id_2"), quantity: 1 }
      ]
    })

## ii. Orders Collection

The orders collection stores order details, including the user who placed the order, items in the order, and total price.

    db.orders.insertOne({
      userId: ObjectId("user_id_1"),
      orderDate: new Date(),
      status: "Shipped",
      totalPrice: 150.75,
      items: [
        { productId: ObjectId("product_id_1"), quantity: 2, price: 50 },
        { productId: ObjectId("product_id_2"), quantity: 1, price: 50.75 }
      ],
      shippingAddress: {
        street: "456 Another St",
        city: "Some City",
        zipCode: "67890"
      }
    })

## iii. Products Collection

The products collection stores product details, including name, description, price, category, stock quantity, and images.

    db.products.insertOne({
      name: "Laptop",
      description: "A powerful gaming laptop",
      price: 999.99,
      category: "Electronics",
      stockQuantity: 50,
      createdAt: new Date(),
      updatedAt: new Date()
    })

Relationships

   Users → Orders: A user can place many orders. This is a one-to-many relationship. The orders field in the users collection references order documents.
   Orders → Products: Each order can contain multiple products. This is a many-to-many relationship, handled by embedding product references (via productId) in the items array within the orders collection.
   Users → Cart: The cart is embedded within the user document. It contains references to products in the form of productId and the quantity of each product.

## MongoDB Queries
# i.  Insert a User
    
    db.users.insertOne({
      username: "john_doe",
      email: "john.doe@example.com",
      passwordHash: "hashed_password",
      fullName: "John Doe",
      address: {
        street: "123 Main St",
        city: "Some City",
        zipCode: "12345"
      },
      phone: "123-456-7890"
    })

## ii. Insert an Order

    db.orders.insertOne({
      userId: ObjectId("user_id_1"),
      orderDate: new Date(),
      status: "Pending",
      totalPrice: 200.99,
      items: [
        { productId: ObjectId("product_id_1"), quantity: 2, price: 50 },
        { productId: ObjectId("product_id_2"), quantity: 1, price: 50.75 }
      ]
    })

## iii. Insert a Product

    db.products.insertOne({
      name: "Wireless Mouse",
      description: "A comfortable wireless mouse",
      price: 20.99,
      category: "Accessories",
      stockQuantity: 100,
    })

## iv. Fetch Orders for a User

    db.users.aggregate([
      {
        $match: { _id: ObjectId("user_id_1") }
      },
      {
        $lookup: {
          from: "orders",
          localField: "orders",
          foreignField: "_id",
          as: "orderDetails"
        }
      }
    ])

##v. Get Products in an Order

    db.orders.aggregate([
      {
        $match: { _id: ObjectId("order_id_1") }
      },
      {
        $lookup: {
          from: "products",
          localField: "items.productId",
          foreignField: "_id",
          as: "orderItems"
        }
      }
    ])

### 9. Indexes

To optimize query performance, create indexes on frequently queried fields:

  Index for faster searching of users by email:

    db.users.createIndex({ email: 1 });

Index for faster searching of orders by userId:

    db.orders.createIndex({ userId: 1 });

 Index for faster searching of products by category:

    db.products.createIndex({ category: 1 });

Aggregation Example
## Aggregate Data

       db.books.aggregate([
          // 1. Find the total number of books per genre
          {
            $facet: {
              totalBooksPerGenre: [
                {
                  $group: {
                    _id: "$genre",
                    totalBooks: { $sum: 1 }
                  }
                }
              ],
          // 2. Calculate the average published year of all books
          averagePublishedYear: [
            {
              $group: {
                _id: null,
                averagePublishedYear: { $avg: "$publishedYear" }
              }
            }
          ],
          // 3. Identify the top-rated book
          topRatedBook: [
            {
              $sort: { rating: -1 }
            },
            {
              $limit: 1
            }
          ]
        }
      }
    ])

## Create an Index on the Author Field

To create an index on the author field:

    db.books.createIndex({ author: 1 });

## Benefits of Indexing in MongoDB
1. Improved Query Performance

Indexes speed up query execution by allowing MongoDB to quickly locate documents based on indexed fields, without scanning the entire collection.
2. Faster Sorting

Indexes allow MongoDB to perform sorting operations more efficiently.
3. Unique Constraints

Indexes can enforce unique constraints, ensuring data integrity (e.g., no duplicate ISBNs).
4. Efficient Aggregation

Indexes can optimize aggregation queries, especially when filtering, sorting, or grouping data.
