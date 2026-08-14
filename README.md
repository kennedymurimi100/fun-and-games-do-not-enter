# fun-and-games-do-not-enter
// FILE: .env
MONGODB_URI=mongodb+srv://<username>:<password>@cluster0.abc123.mongodb.net/communityhub?retryWrites=true&w=majority&appName=Cluster0
PORT=3000


// FILE: models/Post.js

const mongoose = require("mongoose");

const postSchema = new mongoose.Schema(
  {
    title: {
      type: String,
      required: true,
      trim: true
    },
    body: {
      type: String,
      required: true,
      trim: true
    },
    author: {
      type: String,
      required: true,
      trim: true
    },
    published: {
      type: Boolean,
      default: false
    }
  },
  { timestamps: true }
);

module.exports = mongoose.model("Post", postSchema);


// FILE: routes/postRoutes.js

const express = require("express");
const router = express.Router();
const Post = require("../models/Post");

// GET all posts
router.get("/", async (req, res) => {
  try {
    const posts = await Post.find().sort({ createdAt: -1 });
    res.json(posts);
  } catch (error) {
    res.status(500).json({ error: "Failed to fetch posts" });
  }
});

// GET one post by ID
router.get("/:id", async (req, res) => {
  try {
    const post = await Post.findById(req.params.id);

    if (!post) {
      return res.status(404).json({ error: "Post not found" });
    }

    res.json(post);
  } catch (error) {
    res.status(400).json({ error: "Invalid post ID" });
  }
});

// CREATE post
router.post("/", async (req, res) => {
  try {
    const newPost = await Post.create(req.body);
    res.status(201).json(newPost);
  } catch (error) {
    res.status(400).json({ error: "Failed to create post" });
  }
});

// UPDATE post
router.put("/:id", async (req, res) => {
  try {
    const updatedPost = await Post.findByIdAndUpdate(
      req.params.id,
      req.body,
      { new: true, runValidators: true }
    );

    if (!updatedPost) {
      return res.status(404).json({ error: "Post not found" });
    }

    res.json(updatedPost);
  } catch (error) {
    res.status(400).json({ error: "Failed to update post" });
  }
});

// DELETE post
router.delete("/:id", async (req, res) => {
  try {
    const deletedPost = await Post.findByIdAndDelete(req.params.id);

    if (!deletedPost) {
      return res.status(404).json({ error: "Post not found" });
    }

    res.json({ message: "Post deleted successfully" });
  } catch (error) {
    res.status(400).json({ error: "Failed to delete post" });
  }
});

module.exports = router;


// FILE: server.js

require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const postRoutes = require("./routes/postRoutes");

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());

// Simple request logger
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// Connect to MongoDB Atlas
mongoose
  .connect(process.env.MONGODB_URI)
  .then(() => {
    console.log("Connected to MongoDB Atlas");
  })
  .catch((error) => {
    console.error("MongoDB connection error:", error.message);
  });

// Routes
app.get("/", (req, res) => {
  res.json({
    message: "CommunityHub API is running"
  });
});

app.use("/api/posts", postRoutes);

// 404 handler
app.use((req, res) => {
  res.status(404).json({ error: "Route not found" });
});

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});


// FILE: example-requests

// GET all posts
// GET http://localhost:3000/api/posts

// GET one post
// GET http://localhost:3000/api/posts/65f123abc4567890def12345

// CREATE post
// POST http://localhost:3000/api/posts
// Body JSON:
// {
//   "title": "Hello MongoDB",
//   "body": "This is my first database post!",
//   "author": "Anne",
//   "published": true
// }

// UPDATE post
// PUT http://localhost:3000/api/posts/65f123abc4567890def12345
// Body JSON:
// {
//   "title": "Updated Title",
//   "body": "Updated body text",
//   "author": "Anne",
//   "published": false
// }

// DELETE post
// DELETE http://localhost:3000/api/posts/65f123abc4567890def12345
