# Project 1: Blog Database System 📝

**Difficulty:** Beginner | **Estimated Time:** 10-14 hours | **Technologies:** MongoDB, Mongoose, Node.js

## 🎯 Project Overview

Design and implement a comprehensive blog database system using MongoDB. This project demonstrates fundamental MongoDB concepts including document modeling, CRUD operations, indexing, and basic aggregation pipelines. Perfect for understanding how to structure and query document-based data.

## 🚀 What You'll Build

A complete blog database system with the following features:
- ✅ User management with profiles and authentication data
- ✅ Blog posts with rich content and metadata
- ✅ Comment system with nested replies
- ✅ Category and tag management
- ✅ Search functionality with text indexing
- ✅ Analytics and statistics aggregation
- ✅ Content moderation and status management
- ✅ Performance optimization with indexing

## 🛠️ Technical Requirements

### Database Schema Design

#### Users Collection
```javascript
{
  "_id": ObjectId("..."),
  "username": "johndoe",
  "email": "john@example.com",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "bio": "Passionate blogger and developer",
    "avatar": "https://example.com/avatars/john.jpg",
    "website": "https://johndoe.com",
    "location": "New York, USA",
    "socialLinks": {
      "twitter": "@johndoe",
      "linkedin": "johndoe",
      "github": "johndoe"
    }
  },
  "auth": {
    "passwordHash": "hashed_password",
    "emailVerified": true,
    "lastLogin": ISODate("2024-07-20T10:30:00Z"),
    "loginCount": 42
  },
  "preferences": {
    "theme": "dark",
    "emailNotifications": true,
    "newsletter": true
  },
  "stats": {
    "postsCount": 15,
    "commentsCount": 89,
    "followersCount": 234
  },
  "status": "active", // active, suspended, deleted
  "createdAt": ISODate("2024-01-15T08:00:00Z"),
  "updatedAt": ISODate("2024-07-20T10:30:00Z")
}
```

#### Posts Collection
```javascript
{
  "_id": ObjectId("..."),
  "title": "Getting Started with MongoDB",
  "slug": "getting-started-with-mongodb",
  "content": {
    "markdown": "# Getting Started with MongoDB\n\nMongoDB is a document database...",
    "html": "<h1>Getting Started with MongoDB</h1><p>MongoDB is a document database...</p>",
    "excerpt": "Learn the basics of MongoDB, a powerful document database..."
  },
  "author": {
    "userId": ObjectId("..."),
    "username": "johndoe",
    "displayName": "John Doe"
  },
  "metadata": {
    "readingTime": 8, // minutes
    "wordCount": 1200,
    "language": "en"
  },
  "seo": {
    "metaTitle": "Getting Started with MongoDB - Complete Guide",
    "metaDescription": "Learn MongoDB basics with this comprehensive guide...",
    "keywords": ["mongodb", "database", "nosql", "tutorial"]
  },
  "categories": [
    {
      "id": ObjectId("..."),
      "name": "Database",
      "slug": "database"
    }
  ],
  "tags": ["mongodb", "nosql", "database", "tutorial"],
  "status": "published", // draft, published, archived, deleted
  "visibility": "public", // public, private, unlisted
  "featured": false,
  "stats": {
    "views": 1250,
    "likes": 89,
    "shares": 23,
    "commentsCount": 15
  },
  "publishedAt": ISODate("2024-07-15T09:00:00Z"),
  "createdAt": ISODate("2024-07-10T14:30:00Z"),
  "updatedAt": ISODate("2024-07-15T09:00:00Z")
}
```

#### Comments Collection
```javascript
{
  "_id": ObjectId("..."),
  "postId": ObjectId("..."),
  "author": {
    "userId": ObjectId("..."),
    "username": "janedoe",
    "displayName": "Jane Doe",
    "avatar": "https://example.com/avatars/jane.jpg"
  },
  "content": {
    "text": "Great article! Very helpful for beginners.",
    "html": "<p>Great article! Very helpful for beginners.</p>"
  },
  "parentId": null, // null for top-level comments, ObjectId for replies
  "replies": [
    {
      "_id": ObjectId("..."),
      "author": {
        "userId": ObjectId("..."),
        "username": "johndoe",
        "displayName": "John Doe"
      },
      "content": {
        "text": "Thank you! Glad you found it helpful.",
        "html": "<p>Thank you! Glad you found it helpful.</p>"
      },
      "createdAt": ISODate("2024-07-16T11:00:00Z"),
      "updatedAt": ISODate("2024-07-16T11:00:00Z")
    }
  ],
  "stats": {
    "likes": 5,
    "repliesCount": 1
  },
  "status": "approved", // pending, approved, rejected, deleted
  "flagged": false,
  "createdAt": ISODate("2024-07-16T10:30:00Z"),
  "updatedAt": ISODate("2024-07-16T10:30:00Z")
}
```

#### Categories Collection
```javascript
{
  "_id": ObjectId("..."),
  "name": "Database",
  "slug": "database",
  "description": "Articles about databases and data management",
  "color": "#3498db",
  "icon": "database",
  "parentId": null, // for hierarchical categories
  "stats": {
    "postsCount": 25,
    "subscribersCount": 150
  },
  "seo": {
    "metaTitle": "Database Articles",
    "metaDescription": "Explore articles about databases..."
  },
  "status": "active",
  "createdAt": ISODate("2024-01-01T00:00:00Z"),
  "updatedAt": ISODate("2024-07-20T10:00:00Z")
}
```

## 📋 Step-by-Step Implementation Guide

### Phase 1: Database Setup and Connection (2-3 hours)

#### 1. Project Setup
```bash
mkdir blog-database-system
cd blog-database-system
npm init -y

# Install dependencies
npm install mongodb mongoose faker
npm install --save-dev jest

# Create project structure
mkdir src models data scripts
touch src/database.js src/seedData.js
```

#### 2. Database Connection
```javascript
// src/database.js
const mongoose = require('mongoose');

class Database {
  constructor() {
    this.connection = null;
  }

  async connect(uri = 'mongodb://localhost:27017/blog_system') {
    try {
      this.connection = await mongoose.connect(uri, {
        useNewUrlParser: true,
        useUnifiedTopology: true,
      });
      
      console.log('✅ Connected to MongoDB');
      return this.connection;
    } catch (error) {
      console.error('❌ MongoDB connection error:', error);
      throw error;
    }
  }

  async disconnect() {
    if (this.connection) {
      await mongoose.disconnect();
      console.log('📴 Disconnected from MongoDB');
    }
  }

  async dropDatabase() {
    if (this.connection) {
      await mongoose.connection.db.dropDatabase();
      console.log('🗑️ Database dropped');
    }
  }
}

module.exports = Database;
```

### Phase 2: Schema Definition with Mongoose (3-4 hours)

#### 1. User Schema
```javascript
// models/User.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true,
    unique: true,
    trim: true,
    minlength: 3,
    maxlength: 30
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Invalid email']
  },
  profile: {
    firstName: { type: String, required: true, trim: true },
    lastName: { type: String, required: true, trim: true },
    bio: { type: String, maxlength: 500 },
    avatar: { type: String },
    website: { type: String },
    location: { type: String },
    socialLinks: {
      twitter: String,
      linkedin: String,
      github: String
    }
  },
  auth: {
    passwordHash: { type: String, required: true },
    emailVerified: { type: Boolean, default: false },
    lastLogin: Date,
    loginCount: { type: Number, default: 0 }
  },
  preferences: {
    theme: { type: String, enum: ['light', 'dark'], default: 'light' },
    emailNotifications: { type: Boolean, default: true },
    newsletter: { type: Boolean, default: false }
  },
  stats: {
    postsCount: { type: Number, default: 0 },
    commentsCount: { type: Number, default: 0 },
    followersCount: { type: Number, default: 0 }
  },
  status: {
    type: String,
    enum: ['active', 'suspended', 'deleted'],
    default: 'active'
  }
}, {
  timestamps: true,
  toJSON: { virtuals: true },
  toObject: { virtuals: true }
});

// Indexes
userSchema.index({ username: 1 });
userSchema.index({ email: 1 });
userSchema.index({ 'profile.firstName': 1, 'profile.lastName': 1 });
userSchema.index({ status: 1 });

// Virtual for full name
userSchema.virtual('profile.fullName').get(function() {
  return `${this.profile.firstName} ${this.profile.lastName}`;
});

// Methods
userSchema.methods.toPublicJSON = function() {
  const user = this.toObject();
  delete user.auth;
  return user;
};

userSchema.methods.incrementPostCount = function() {
  this.stats.postsCount += 1;
  return this.save();
};

module.exports = mongoose.model('User', userSchema);
```

#### 2. Post Schema
```javascript
// models/Post.js
const mongoose = require('mongoose');

const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true,
    trim: true,
    maxlength: 200
  },
  slug: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  content: {
    markdown: { type: String, required: true },
    html: { type: String, required: true },
    excerpt: { type: String, maxlength: 300 }
  },
  author: {
    userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    username: { type: String, required: true },
    displayName: { type: String, required: true }
  },
  metadata: {
    readingTime: { type: Number, default: 0 },
    wordCount: { type: Number, default: 0 },
    language: { type: String, default: 'en' }
  },
  seo: {
    metaTitle: String,
    metaDescription: String,
    keywords: [String]
  },
  categories: [{
    id: { type: mongoose.Schema.Types.ObjectId, ref: 'Category' },
    name: String,
    slug: String
  }],
  tags: [{ type: String, lowercase: true, trim: true }],
  status: {
    type: String,
    enum: ['draft', 'published', 'archived', 'deleted'],
    default: 'draft'
  },
  visibility: {
    type: String,
    enum: ['public', 'private', 'unlisted'],
    default: 'public'
  },
  featured: { type: Boolean, default: false },
  stats: {
    views: { type: Number, default: 0 },
    likes: { type: Number, default: 0 },
    shares: { type: Number, default: 0 },
    commentsCount: { type: Number, default: 0 }
  },
  publishedAt: Date
}, {
  timestamps: true
});

// Indexes
postSchema.index({ slug: 1 });
postSchema.index({ 'author.userId': 1 });
postSchema.index({ status: 1, visibility: 1 });
postSchema.index({ publishedAt: -1 });
postSchema.index({ featured: 1, publishedAt: -1 });
postSchema.index({ tags: 1 });
postSchema.index({ 'categories.slug': 1 });

// Text index for search
postSchema.index({
  title: 'text',
  'content.markdown': 'text',
  tags: 'text'
}, {
  weights: {
    title: 10,
    tags: 5,
    'content.markdown': 1
  }
});

// Pre-save middleware
postSchema.pre('save', function(next) {
  if (this.isModified('content.markdown')) {
    // Calculate reading time (average 200 words per minute)
    const wordCount = this.content.markdown.split(/\s+/).length;
    this.metadata.wordCount = wordCount;
    this.metadata.readingTime = Math.ceil(wordCount / 200);
  }
  
  if (this.isModified('status') && this.status === 'published' && !this.publishedAt) {
    this.publishedAt = new Date();
  }
  
  next();
});

// Methods
postSchema.methods.incrementViews = function() {
  this.stats.views += 1;
  return this.save();
};

postSchema.methods.addLike = function() {
  this.stats.likes += 1;
  return this.save();
};

module.exports = mongoose.model('Post', postSchema);
```

### Phase 3: CRUD Operations and Queries (3-4 hours)

#### 1. Blog Service Class
```javascript
// src/blogService.js
const User = require('../models/User');
const Post = require('../models/Post');
const Comment = require('../models/Comment');
const Category = require('../models/Category');

class BlogService {
  // User operations
  async createUser(userData) {
    try {
      const user = new User(userData);
      await user.save();
      return user.toPublicJSON();
    } catch (error) {
      if (error.code === 11000) {
        throw new Error('Username or email already exists');
      }
      throw error;
    }
  }

  async getUserByUsername(username) {
    return await User.findOne({ username, status: 'active' })
      .select('-auth.passwordHash');
  }

  // Post operations
  async createPost(postData) {
    const post = new Post(postData);
    await post.save();
    
    // Update user's post count
    await User.findByIdAndUpdate(
      postData.author.userId,
      { $inc: { 'stats.postsCount': 1 } }
    );
    
    return post;
  }

  async getPublishedPosts(options = {}) {
    const {
      page = 1,
      limit = 10,
      category,
      tag,
      author,
      featured,
      sortBy = 'publishedAt',
      sortOrder = -1
    } = options;

    const query = {
      status: 'published',
      visibility: 'public'
    };

    if (category) query['categories.slug'] = category;
    if (tag) query.tags = tag;
    if (author) query['author.username'] = author;
    if (featured !== undefined) query.featured = featured;

    const posts = await Post.find(query)
      .sort({ [sortBy]: sortOrder })
      .skip((page - 1) * limit)
      .limit(limit)
      .populate('author.userId', 'profile.firstName profile.lastName profile.avatar');

    const total = await Post.countDocuments(query);

    return {
      posts,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    };
  }

  async searchPosts(searchTerm, options = {}) {
    const { page = 1, limit = 10 } = options;

    const posts = await Post.find(
      {
        $text: { $search: searchTerm },
        status: 'published',
        visibility: 'public'
      },
      { score: { $meta: 'textScore' } }
    )
    .sort({ score: { $meta: 'textScore' } })
    .skip((page - 1) * limit)
    .limit(limit);

    return posts;
  }

  // Analytics and aggregations
  async getPostStats() {
    const stats = await Post.aggregate([
      {
        $group: {
          _id: null,
          totalPosts: { $sum: 1 },
          publishedPosts: {
            $sum: { $cond: [{ $eq: ['$status', 'published'] }, 1, 0] }
          },
          draftPosts: {
            $sum: { $cond: [{ $eq: ['$status', 'draft'] }, 1, 0] }
          },
          totalViews: { $sum: '$stats.views' },
          totalLikes: { $sum: '$stats.likes' },
          avgReadingTime: { $avg: '$metadata.readingTime' }
        }
      }
    ]);

    return stats[0] || {};
  }

  async getTopAuthors(limit = 10) {
    return await Post.aggregate([
      { $match: { status: 'published' } },
      {
        $group: {
          _id: '$author.userId',
          username: { $first: '$author.username' },
          displayName: { $first: '$author.displayName' },
          postsCount: { $sum: 1 },
          totalViews: { $sum: '$stats.views' },
          totalLikes: { $sum: '$stats.likes' }
        }
      },
      { $sort: { postsCount: -1 } },
      { $limit: limit }
    ]);
  }

  async getPopularTags(limit = 20) {
    return await Post.aggregate([
      { $match: { status: 'published' } },
      { $unwind: '$tags' },
      {
        $group: {
          _id: '$tags',
          count: { $sum: 1 },
          totalViews: { $sum: '$stats.views' }
        }
      },
      { $sort: { count: -1 } },
      { $limit: limit },
      {
        $project: {
          tag: '$_id',
          count: 1,
          totalViews: 1,
          _id: 0
        }
      }
    ]);
  }

  async getMonthlyPostStats() {
    return await Post.aggregate([
      { $match: { status: 'published' } },
      {
        $group: {
          _id: {
            year: { $year: '$publishedAt' },
            month: { $month: '$publishedAt' }
          },
          count: { $sum: 1 },
          totalViews: { $sum: '$stats.views' }
        }
      },
      { $sort: { '_id.year': -1, '_id.month': -1 } },
      { $limit: 12 }
    ]);
  }
}

module.exports = BlogService;
```

### Phase 4: Data Seeding and Testing (2-3 hours)

#### 1. Data Seeding Script
```javascript
// src/seedData.js
const faker = require('faker');
const Database = require('./database');
const User = require('../models/User');
const Post = require('../models/Post');
const Category = require('../models/Category');

class DataSeeder {
  constructor() {
    this.db = new Database();
  }

  async seed() {
    try {
      await this.db.connect();
      await this.db.dropDatabase();
      
      console.log('🌱 Seeding database...');
      
      const categories = await this.seedCategories();
      const users = await this.seedUsers(10);
      const posts = await this.seedPosts(users, categories, 50);
      
      console.log('✅ Database seeded successfully!');
      console.log(`Created ${users.length} users, ${posts.length} posts, ${categories.length} categories`);
      
    } catch (error) {
      console.error('❌ Seeding failed:', error);
    } finally {
      await this.db.disconnect();
    }
  }

  async seedCategories() {
    const categoryData = [
      { name: 'Technology', slug: 'technology', description: 'Tech articles and tutorials', color: '#3498db' },
      { name: 'Programming', slug: 'programming', description: 'Programming guides and tips', color: '#e74c3c' },
      { name: 'Database', slug: 'database', description: 'Database design and optimization', color: '#2ecc71' },
      { name: 'Web Development', slug: 'web-development', description: 'Frontend and backend development', color: '#f39c12' },
      { name: 'DevOps', slug: 'devops', description: 'Deployment and infrastructure', color: '#9b59b6' }
    ];

    const categories = await Category.insertMany(categoryData);
    console.log(`📁 Created ${categories.length} categories`);
    return categories;
  }

  async seedUsers(count) {
    const users = [];
    
    for (let i = 0; i < count; i++) {
      const userData = {
        username: faker.internet.userName().toLowerCase(),
        email: faker.internet.email().toLowerCase(),
        profile: {
          firstName: faker.name.firstName(),
          lastName: faker.name.lastName(),
          bio: faker.lorem.sentences(2),
          avatar: faker.internet.avatar(),
          website: faker.internet.url(),
          location: `${faker.address.city()}, ${faker.address.country()}`,
          socialLinks: {
            twitter: faker.internet.userName(),
            github: faker.internet.userName(),
            linkedin: faker.internet.userName()
          }
        },
        auth: {
          passwordHash: 'hashed_password_here',
          emailVerified: faker.datatype.boolean(),
          lastLogin: faker.date.recent(),
          loginCount: faker.datatype.number({ min: 1, max: 100 })
        },
        preferences: {
          theme: faker.random.arrayElement(['light', 'dark']),
          emailNotifications: faker.datatype.boolean(),
          newsletter: faker.datatype.boolean()
        }
      };
      
      users.push(userData);
    }
    
    const createdUsers = await User.insertMany(users);
    console.log(`👥 Created ${createdUsers.length} users`);
    return createdUsers;
  }

  async seedPosts(users, categories, count) {
    const posts = [];
    
    for (let i = 0; i < count; i++) {
      const author = faker.random.arrayElement(users);
      const selectedCategories = faker.random.arrayElements(categories, faker.datatype.number({ min: 1, max: 3 }));
      const title = faker.lorem.sentence();
      const slug = title.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g, '');
      
      const postData = {
        title,
        slug: `${slug}-${Date.now()}-${i}`,
        content: {
          markdown: faker.lorem.paragraphs(faker.datatype.number({ min: 3, max: 10 })),
          html: `<p>${faker.lorem.paragraphs(faker.datatype.number({ min: 3, max: 10 }), '</p><p>')}</p>`,
          excerpt: faker.lorem.sentences(2)
        },
        author: {
          userId: author._id,
          username: author.username,
          displayName: author.profile.fullName
        },
        categories: selectedCategories.map(cat => ({
          id: cat._id,
          name: cat.name,
          slug: cat.slug
        })),
        tags: faker.random.words(faker.datatype.number({ min: 2, max: 6 })).split(' '),
        status: faker.random.arrayElement(['draft', 'published', 'published', 'published']), // More published posts
        featured: faker.datatype.boolean() && faker.datatype.number({ min: 1, max: 10 }) > 8,
        stats: {
          views: faker.datatype.number({ min: 0, max: 5000 }),
          likes: faker.datatype.number({ min: 0, max: 200 }),
          shares: faker.datatype.number({ min: 0, max: 50 })
        },
        publishedAt: faker.date.past()
      };
      
      posts.push(postData);
    }
    
    const createdPosts = await Post.insertMany(posts);
    console.log(`📝 Created ${createdPosts.length} posts`);
    return createdPosts;
  }
}

// Run seeding if called directly
if (require.main === module) {
  const seeder = new DataSeeder();
  seeder.seed();
}

module.exports = DataSeeder;
```

## 🎯 Advanced Queries and Operations

### Complex Aggregation Examples
```javascript
// Get trending posts (high engagement in last 7 days)
async getTrendingPosts() {
  const sevenDaysAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
  
  return await Post.aggregate([
    {
      $match: {
        status: 'published',
        publishedAt: { $gte: sevenDaysAgo }
      }
    },
    {
      $addFields: {
        engagementScore: {
          $add: [
            { $multiply: ['$stats.views', 1] },
            { $multiply: ['$stats.likes', 5] },
            { $multiply: ['$stats.shares', 10] }
          ]
        }
      }
    },
    { $sort: { engagementScore: -1 } },
    { $limit: 10 }
  ]);
}

// Get author performance analytics
async getAuthorAnalytics(authorId) {
  return await Post.aggregate([
    { $match: { 'author.userId': authorId, status: 'published' } },
    {
      $group: {
        _id: null,
        totalPosts: { $sum: 1 },
        totalViews: { $sum: '$stats.views' },
        totalLikes: { $sum: '$stats.likes' },
        avgViews: { $avg: '$stats.views' },
        avgLikes: { $avg: '$stats.likes' },
        avgReadingTime: { $avg: '$metadata.readingTime' },
        mostPopularPost: { $max: '$stats.views' }
      }
    }
  ]);
}
```

## 📚 Learning Outcomes

After completing this project, you will have:
- Designed complex document schemas with relationships
- Implemented efficient indexing strategies
- Built aggregation pipelines for analytics
- Handled data validation and constraints
- Created search functionality with text indexes
- Understood MongoDB performance optimization
- Worked with embedded vs referenced data patterns

---

**Ready to build your blog database?** This project will give you hands-on experience with MongoDB fundamentals and prepare you for more complex database operations.

**Next Project:** [E-commerce Database](../02-ecommerce-database/README.md) - Learn advanced data modeling and complex relationships.

---

*This project demonstrates essential MongoDB skills that employers look for in database developers. Focus on clean schema design and efficient queries!*
