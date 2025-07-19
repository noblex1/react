# MongoDB Essentials Cheat Sheet 🍃

## Basic Operations

### Database Operations
```javascript
// Show databases
show dbs

// Use/create database
use myapp

// Show current database
db

// Drop database
db.dropDatabase()

// Show collections
show collections
```

### Collection Operations
```javascript
// Create collection
db.createCollection("users")

// Drop collection
db.users.drop()

// Get collection stats
db.users.stats()
```

## CRUD Operations

### Create (Insert)
```javascript
// Insert one document
db.users.insertOne({
  name: "John Doe",
  email: "john@example.com",
  age: 30
})

// Insert multiple documents
db.users.insertMany([
  { name: "Jane Smith", email: "jane@example.com", age: 25 },
  { name: "Bob Johnson", email: "bob@example.com", age: 35 }
])

// Insert with custom _id
db.users.insertOne({
  _id: "custom-id-123",
  name: "Alice Brown",
  email: "alice@example.com"
})
```

### Read (Find)
```javascript
// Find all documents
db.users.find()

// Find with pretty formatting
db.users.find().pretty()

// Find one document
db.users.findOne()

// Find with query
db.users.find({ age: 30 })

// Find with multiple conditions
db.users.find({ age: 30, name: "John Doe" })

// Find with projection (select specific fields)
db.users.find({}, { name: 1, email: 1, _id: 0 })

// Find with limit and skip
db.users.find().limit(5).skip(10)

// Find with sorting
db.users.find().sort({ age: 1 }) // 1 for ascending, -1 for descending
```

### Update
```javascript
// Update one document
db.users.updateOne(
  { name: "John Doe" },
  { $set: { age: 31 } }
)

// Update multiple documents
db.users.updateMany(
  { age: { $lt: 30 } },
  { $set: { category: "young" } }
)

// Replace entire document
db.users.replaceOne(
  { name: "John Doe" },
  { name: "John Doe", email: "newemail@example.com", age: 32 }
)

// Upsert (update or insert)
db.users.updateOne(
  { email: "new@example.com" },
  { $set: { name: "New User", age: 25 } },
  { upsert: true }
)
```

### Delete
```javascript
// Delete one document
db.users.deleteOne({ name: "John Doe" })

// Delete multiple documents
db.users.deleteMany({ age: { $lt: 18 } })

// Delete all documents in collection
db.users.deleteMany({})
```

## Query Operators

### Comparison Operators
```javascript
// Equal
db.users.find({ age: 30 })

// Not equal
db.users.find({ age: { $ne: 30 } })

// Greater than
db.users.find({ age: { $gt: 25 } })

// Greater than or equal
db.users.find({ age: { $gte: 25 } })

// Less than
db.users.find({ age: { $lt: 35 } })

// Less than or equal
db.users.find({ age: { $lte: 35 } })

// In array
db.users.find({ age: { $in: [25, 30, 35] } })

// Not in array
db.users.find({ age: { $nin: [25, 30, 35] } })
```

### Logical Operators
```javascript
// AND (implicit)
db.users.find({ age: 30, name: "John Doe" })

// AND (explicit)
db.users.find({ $and: [{ age: 30 }, { name: "John Doe" }] })

// OR
db.users.find({ $or: [{ age: 30 }, { name: "John Doe" }] })

// NOT
db.users.find({ age: { $not: { $gt: 30 } } })

// NOR
db.users.find({ $nor: [{ age: 30 }, { name: "John Doe" }] })
```

### Element Operators
```javascript
// Field exists
db.users.find({ email: { $exists: true } })

// Field doesn't exist
db.users.find({ phone: { $exists: false } })

// Type check
db.users.find({ age: { $type: "number" } })
db.users.find({ age: { $type: 16 } }) // 16 is BSON type for 32-bit integer
```

### Array Operators
```javascript
// Array contains element
db.posts.find({ tags: "mongodb" })

// Array contains all elements
db.posts.find({ tags: { $all: ["mongodb", "database"] } })

// Array size
db.posts.find({ tags: { $size: 3 } })

// Element match
db.posts.find({
  comments: {
    $elemMatch: { author: "John", rating: { $gte: 4 } }
  }
})
```

### String Operators
```javascript
// Regular expression
db.users.find({ name: { $regex: /^John/ } })
db.users.find({ name: { $regex: "john", $options: "i" } }) // case insensitive

// Text search (requires text index)
db.posts.find({ $text: { $search: "mongodb tutorial" } })
```

## Update Operators

### Field Update Operators
```javascript
// Set field value
db.users.updateOne({ _id: 1 }, { $set: { age: 31 } })

// Unset (remove) field
db.users.updateOne({ _id: 1 }, { $unset: { age: "" } })

// Increment numeric value
db.users.updateOne({ _id: 1 }, { $inc: { age: 1 } })

// Multiply numeric value
db.users.updateOne({ _id: 1 }, { $mul: { age: 1.1 } })

// Rename field
db.users.updateOne({ _id: 1 }, { $rename: { "name": "fullName" } })

// Set minimum value
db.users.updateOne({ _id: 1 }, { $min: { age: 18 } })

// Set maximum value
db.users.updateOne({ _id: 1 }, { $max: { age: 65 } })

// Set current date
db.users.updateOne({ _id: 1 }, { $currentDate: { lastModified: true } })
```

### Array Update Operators
```javascript
// Add to array
db.posts.updateOne({ _id: 1 }, { $push: { tags: "new-tag" } })

// Add multiple to array
db.posts.updateOne({ _id: 1 }, { $push: { tags: { $each: ["tag1", "tag2"] } } })

// Add to array if not exists
db.posts.updateOne({ _id: 1 }, { $addToSet: { tags: "unique-tag" } })

// Remove from array
db.posts.updateOne({ _id: 1 }, { $pull: { tags: "old-tag" } })

// Remove multiple from array
db.posts.updateOne({ _id: 1 }, { $pullAll: { tags: ["tag1", "tag2"] } })

// Remove first/last element
db.posts.updateOne({ _id: 1 }, { $pop: { tags: 1 } }) // 1 for last, -1 for first

// Update array element by position
db.posts.updateOne({ _id: 1 }, { $set: { "tags.0": "updated-tag" } })

// Update array element by condition
db.posts.updateOne(
  { _id: 1, "comments.author": "John" },
  { $set: { "comments.$.approved": true } }
)
```

## Indexing

### Create Indexes
```javascript
// Single field index
db.users.createIndex({ email: 1 }) // 1 for ascending, -1 for descending

// Compound index
db.users.createIndex({ name: 1, age: -1 })

// Text index
db.posts.createIndex({ title: "text", content: "text" })

// Partial index
db.users.createIndex(
  { email: 1 },
  { partialFilterExpression: { age: { $gte: 18 } } }
)

// Unique index
db.users.createIndex({ email: 1 }, { unique: true })

// Sparse index (only indexes documents with the field)
db.users.createIndex({ phone: 1 }, { sparse: true })

// TTL index (time to live)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })
```

### Index Management
```javascript
// List indexes
db.users.getIndexes()

// Drop index
db.users.dropIndex({ email: 1 })
db.users.dropIndex("email_1") // by name

// Drop all indexes (except _id)
db.users.dropIndexes()

// Reindex collection
db.users.reIndex()

// Get index stats
db.users.stats().indexSizes
```

## Aggregation Framework

### Basic Aggregation Pipeline
```javascript
db.orders.aggregate([
  { $match: { status: "completed" } },
  { $group: { _id: "$customerId", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])
```

### Common Pipeline Stages
```javascript
// Match (filter)
{ $match: { age: { $gte: 18 } } }

// Group
{ $group: { _id: "$category", count: { $sum: 1 }, avgPrice: { $avg: "$price" } } }

// Sort
{ $sort: { createdAt: -1 } }

// Limit
{ $limit: 10 }

// Skip
{ $skip: 20 }

// Project (select/transform fields)
{ $project: { name: 1, email: 1, fullName: { $concat: ["$firstName", " ", "$lastName"] } } }

// Unwind (flatten arrays)
{ $unwind: "$tags" }

// Lookup (join)
{
  $lookup: {
    from: "users",
    localField: "userId",
    foreignField: "_id",
    as: "user"
  }
}

// Add fields
{ $addFields: { totalWithTax: { $multiply: ["$total", 1.1] } } }
```

### Aggregation Operators
```javascript
// Arithmetic
{ $add: ["$price", "$tax"] }
{ $subtract: ["$total", "$discount"] }
{ $multiply: ["$quantity", "$price"] }
{ $divide: ["$total", "$quantity"] }

// Array
{ $size: "$tags" }
{ $arrayElemAt: ["$tags", 0] }
{ $slice: ["$tags", 3] }

// Conditional
{ $cond: { if: { $gte: ["$age", 18] }, then: "adult", else: "minor" } }
{ $ifNull: ["$phone", "N/A"] }

// Date
{ $year: "$createdAt" }
{ $month: "$createdAt" }
{ $dayOfWeek: "$createdAt" }

// String
{ $concat: ["$firstName", " ", "$lastName"] }
{ $substr: ["$name", 0, 3] }
{ $toLower: "$email" }
{ $toUpper: "$name" }
```

## Data Modeling Patterns

### Embedded Documents
```javascript
// User with embedded address
{
  _id: ObjectId("..."),
  name: "John Doe",
  email: "john@example.com",
  address: {
    street: "123 Main St",
    city: "New York",
    zipCode: "10001",
    country: "USA"
  }
}
```

### Referenced Documents
```javascript
// User document
{
  _id: ObjectId("507f1f77bcf86cd799439011"),
  name: "John Doe",
  email: "john@example.com"
}

// Post document with user reference
{
  _id: ObjectId("507f1f77bcf86cd799439012"),
  title: "My Blog Post",
  content: "...",
  authorId: ObjectId("507f1f77bcf86cd799439011")
}
```

### One-to-Many Relationships
```javascript
// Blog post with embedded comments
{
  _id: ObjectId("..."),
  title: "My Post",
  content: "...",
  comments: [
    {
      _id: ObjectId("..."),
      author: "Jane",
      text: "Great post!",
      date: ISODate("2024-07-20")
    }
  ]
}
```

## Performance and Optimization

### Query Performance
```javascript
// Explain query execution
db.users.find({ age: 30 }).explain("executionStats")

// Create index hint
db.users.find({ name: "John", age: 30 }).hint({ name: 1 })

// Profile slow queries
db.setProfilingLevel(2, { slowms: 100 })
db.system.profile.find().sort({ ts: -1 }).limit(5)
```

### Bulk Operations
```javascript
// Bulk insert
db.users.insertMany([
  { name: "User1", email: "user1@example.com" },
  { name: "User2", email: "user2@example.com" }
], { ordered: false })

// Bulk write operations
db.users.bulkWrite([
  { insertOne: { document: { name: "New User" } } },
  { updateOne: { filter: { name: "John" }, update: { $set: { age: 31 } } } },
  { deleteOne: { filter: { name: "Old User" } } }
])
```

## MongoDB with Node.js (Mongoose)

### Connection
```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/myapp', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});
```

### Schema Definition
```javascript
const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, min: 0, max: 120 },
  createdAt: { type: Date, default: Date.now }
});

const User = mongoose.model('User', userSchema);
```

### CRUD with Mongoose
```javascript
// Create
const user = new User({ name: 'John', email: 'john@example.com' });
await user.save();

// Read
const users = await User.find({ age: { $gte: 18 } });
const user = await User.findById(userId);

// Update
await User.findByIdAndUpdate(userId, { age: 31 });

// Delete
await User.findByIdAndDelete(userId);
```

## Useful Commands

### Database Administration
```javascript
// Server status
db.serverStatus()

// Database stats
db.stats()

// Collection stats
db.users.stats()

// Current operations
db.currentOp()

// Kill operation
db.killOp(opId)
```

### Backup and Restore
```bash
# Backup database
mongodump --db myapp --out /backup/

# Restore database
mongorestore --db myapp /backup/myapp/

# Export collection to JSON
mongoexport --db myapp --collection users --out users.json

# Import JSON to collection
mongoimport --db myapp --collection users --file users.json
```

---

**Keep this cheat sheet handy while working with MongoDB!** 🍃
