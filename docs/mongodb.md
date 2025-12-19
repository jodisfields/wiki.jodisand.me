# MongoDB

MongoDB is a NoSQL document database that stores data in flexible, JSON-like documents.

## Installation

```bash
# Install MongoDB Community Edition
# macOS
brew tap mongodb/brew
brew install mongodb-community

# Start MongoDB
brew services start mongodb-community

# Or run manually
mongod --config /usr/local/etc/mongod.conf
```

## MongoDB Shell

```bash
# Connect to MongoDB
mongosh

# Show databases
show dbs

# Use/create database
use mydb

# Show collections
show collections

# Exit
exit
```

## Basic Operations

### Insert Documents

```javascript
// Insert one document
db.users.insertOne({
    name: "John Doe",
    email: "john@example.com",
    age: 30
});

// Insert multiple documents
db.users.insertMany([
    { name: "Alice", email: "alice@example.com", age: 25 },
    { name: "Bob", email: "bob@example.com", age: 35 }
]);
```

### Find Documents

```javascript
// Find all
db.users.find();

// Find with query
db.users.find({ age: 30 });

// Find one
db.users.findOne({ email: "john@example.com" });

// Find with conditions
db.users.find({ age: { $gt: 25 } });
db.users.find({ age: { $gte: 30, $lte: 40 } });
db.users.find({ name: { $in: ["Alice", "Bob"] } });

// Find with projection
db.users.find({}, { name: 1, email: 1, _id: 0 });

// Limit and skip
db.users.find().limit(10);
db.users.find().skip(5).limit(10);

// Sort
db.users.find().sort({ age: 1 });  // Ascending
db.users.find().sort({ age: -1 }); // Descending
```

### Update Documents

```javascript
// Update one
db.users.updateOne(
    { name: "John Doe" },
    { $set: { age: 31 } }
);

// Update many
db.users.updateMany(
    { age: { $lt: 30 } },
    { $set: { status: "young" } }
);

// Replace one
db.users.replaceOne(
    { name: "John Doe" },
    { name: "John Doe", email: "newemail@example.com", age: 31 }
);

// Increment
db.users.updateOne(
    { name: "John Doe" },
    { $inc: { age: 1 } }
);

// Add to array
db.users.updateOne(
    { name: "John Doe" },
    { $push: { hobbies: "reading" } }
);

// Remove from array
db.users.updateOne(
    { name: "John Doe" },
    { $pull: { hobbies: "reading" } }
);

// Upsert (update or insert)
db.users.updateOne(
    { email: "new@example.com" },
    { $set: { name: "New User", age: 25 } },
    { upsert: true }
);
```

### Delete Documents

```javascript
// Delete one
db.users.deleteOne({ name: "John Doe" });

// Delete many
db.users.deleteMany({ age: { $lt: 25 } });

// Delete all
db.users.deleteMany({});
```

## Query Operators

```javascript
// Comparison
db.users.find({ age: { $eq: 30 } });      // Equal
db.users.find({ age: { $ne: 30 } });      // Not equal
db.users.find({ age: { $gt: 30 } });      // Greater than
db.users.find({ age: { $gte: 30 } });     // Greater than or equal
db.users.find({ age: { $lt: 30 } });      // Less than
db.users.find({ age: { $lte: 30 } });     // Less than or equal
db.users.find({ name: { $in: ["Alice", "Bob"] } }); // In array
db.users.find({ name: { $nin: ["Alice", "Bob"] } }); // Not in array

// Logical
db.users.find({
    $and: [
        { age: { $gte: 25 } },
        { age: { $lte: 35 } }
    ]
});

db.users.find({
    $or: [
        { age: { $lt: 25 } },
        { age: { $gt: 35 } }
    ]
});

db.users.find({ age: { $not: { $lt: 30 } } });

// Element
db.users.find({ email: { $exists: true } });
db.users.find({ age: { $type: "int" } });

// Array
db.posts.find({ tags: { $all: ["mongodb", "database"] } });
db.posts.find({ tags: { $size: 3 } });

// Text search
db.articles.createIndex({ content: "text" });
db.articles.find({ $text: { $search: "mongodb tutorial" } });

// Regex
db.users.find({ name: { $regex: /^John/i } });
```

## Aggregation

```javascript
// Basic aggregation
db.orders.aggregate([
    { $match: { status: "completed" } },
    { $group: {
        _id: "$customerId",
        total: { $sum: "$amount" }
    }},
    { $sort: { total: -1 } }
]);

// $project - reshape documents
db.users.aggregate([
    { $project: {
        name: 1,
        email: 1,
        yearOfBirth: { $subtract: [2024, "$age"] }
    }}
]);

// $lookup - join collections
db.orders.aggregate([
    { $lookup: {
        from: "products",
        localField: "productId",
        foreignField: "_id",
        as: "productDetails"
    }}
]);

// $unwind - deconstruct array
db.posts.aggregate([
    { $unwind: "$tags" },
    { $group: {
        _id: "$tags",
        count: { $sum: 1 }
    }}
]);

// $limit and $skip
db.users.aggregate([
    { $sort: { age: -1 } },
    { $skip: 10 },
    { $limit: 5 }
]);
```

## Indexes

```javascript
// Create index
db.users.createIndex({ email: 1 });

// Compound index
db.users.createIndex({ name: 1, age: -1 });

// Unique index
db.users.createIndex({ email: 1 }, { unique: true });

// Text index
db.articles.createIndex({ title: "text", content: "text" });

// Show indexes
db.users.getIndexes();

// Drop index
db.users.dropIndex("email_1");
```

## Data Modeling

```javascript
// Embedded documents
db.users.insertOne({
    name: "John Doe",
    email: "john@example.com",
    address: {
        street: "123 Main St",
        city: "New York",
        zipCode: "10001"
    },
    orders: [
        { orderId: 1, amount: 100, date: new Date() },
        { orderId: 2, amount: 200, date: new Date() }
    ]
});

// References
db.users.insertOne({
    _id: ObjectId("..."),
    name: "John Doe",
    email: "john@example.com"
});

db.orders.insertOne({
    orderId: 1,
    userId: ObjectId("..."),
    amount: 100,
    date: new Date()
});
```

## MongoDB with Node.js

```javascript
const { MongoClient } = require('mongodb');

const url = 'mongodb://localhost:27017';
const client = new MongoClient(url);

async function main() {
    await client.connect();
    console.log('Connected to MongoDB');

    const db = client.db('mydb');
    const collection = db.collection('users');

    // Insert
    const result = await collection.insertOne({
        name: 'John Doe',
        email: 'john@example.com',
        age: 30
    });
    console.log('Inserted:', result.insertedId);

    // Find
    const users = await collection.find({}).toArray();
    console.log('Users:', users);

    // Update
    await collection.updateOne(
        { name: 'John Doe' },
        { $set: { age: 31 } }
    );

    // Delete
    await collection.deleteOne({ name: 'John Doe' });

    await client.close();
}

main().catch(console.error);
```

## Mongoose (ODM)

```javascript
const mongoose = require('mongoose');

// Connect
mongoose.connect('mongodb://localhost:27017/mydb');

// Define schema
const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    age: Number,
    createdAt: { type: Date, default: Date.now }
});

// Create model
const User = mongoose.model('User', userSchema);

// Create document
const user = new User({
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
});

await user.save();

// Find
const users = await User.find();
const user = await User.findOne({ email: 'john@example.com' });
const user = await User.findById(id);

// Update
await User.updateOne({ _id: id }, { age: 31 });
const user = await User.findByIdAndUpdate(id, { age: 31 }, { new: true });

// Delete
await User.deleteOne({ _id: id });
const user = await User.findByIdAndDelete(id);

// Validation
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Name is required'],
        minlength: 3,
        maxlength: 50
    },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true,
        validate: {
            validator: function(v) {
                return /\S+@\S+\.\S+/.test(v);
            },
            message: 'Invalid email format'
        }
    },
    age: {
        type: Number,
        min: 0,
        max: 120
    }
});
```

## Transactions

```javascript
const session = client.startSession();

try {
    await session.withTransaction(async () => {
        await collection1.insertOne({ ... }, { session });
        await collection2.updateOne({ ... }, { ... }, { session });
    });
} finally {
    await session.endSession();
}
```

## Backup and Restore

```bash
# Backup database
mongodump --db mydb --out /backup/

# Backup specific collection
mongodump --db mydb --collection users --out /backup/

# Restore database
mongorestore --db mydb /backup/mydb/

# Export to JSON
mongoexport --db mydb --collection users --out users.json

# Import from JSON
mongoimport --db mydb --collection users --file users.json
```

## Resources

- [Official Documentation](https://docs.mongodb.com/)
- [MongoDB University](https://university.mongodb.com/)
- [Mongoose Documentation](https://mongoosejs.com/)
