# Express.js Routes and Controllers: Category Notes

---

## Step 1: Folder Structure

```
notes-api/
├── controllers/
│   └── categoryController.js
├── models/
│   └── Category.js
├── routes/
│   └── categoryRoutes.js
├── index.js
└── package.json
```

---

## Step 2: Install Packages

```bash
npm init -y
npm install express mongoose
npm install --save-dev nodemon
```

Add the `dev` script in `package.json`:

```json
"scripts": {
  "dev": "nodemon index.js"
}
```

---

## Step 3: Create the Category Model

Open `models/Category.js`:

```javascript
const mongoose = require("mongoose");

const categorySchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      unique: true,
      trim: true,
    },
    description: {
      type: String,
      default: "",
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model("Category", categorySchema);
```

`timestamps: true` adds `createdAt` and `updatedAt` automatically.

---

## Step 4: Create the Controller

Open `controllers/categoryController.js` and import the model:

```javascript
const Category = require("../models/Category");
```

---

## Step 5: Create Category

```javascript
const createCategory = async (req, res) => {
  try {
    const category = await Category.create(req.body);
    res.status(201).json(category);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};
```

---

## Step 6: Get All Categories

```javascript
const getCategories = async (req, res) => {
  try {
    const categories = await Category.find();
    res.json(categories);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};
```

---

## Step 7: Get One Category by ID

```javascript
const getCategoryById = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);

    if (!category) {
      return res.status(404).json({ message: "Category not found" });
    }

    res.json(category);
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
};
```

---

## Step 8: Update Category

```javascript
const updateCategory = async (req, res) => {
  try {
    const category = await Category.findById(req.params.id);

    if (!category) {
      return res.status(404).json({ message: "Category not found" });
    }

    await Category.updateOne({ _id: req.params.id }, req.body);

    ///-------------or-------------------

    // const { name, description } = req.body;
    // await Category.updateOne(
    //   { _id: req.params.id },
    //   { $set: { name, description } }
    // );

    res.json({ message: "Category updated" });
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
};
```

---

## Step 9: Delete Category

```javascript
const deleteCategory = async (req, res) => {
  try {
    const category = await Category.findByIdAndDelete(req.params.id);

    if (!category) {
      return res.status(404).json({ message: "Category not found" });
    }

    res.json({ message: "Category deleted successfully" });
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
};
```

---

## Step 10: Export the Functions

At the end of `categoryController.js`:

```javascript
module.exports = {
  createCategory,
  getCategories,
  getCategoryById,
  updateCategory,
  deleteCategory,
};
```

---

## Step 11: Create the Routes

Open `routes/categoryRoutes.js`:

```javascript
const express = require("express");
const {
  createCategory,
  getCategories,
  getCategoryById,
  updateCategory,
  deleteCategory,
} = require("../controllers/categoryController");

const router = express.Router();

router.post("/", createCategory);
router.get("/", getCategories);
router.get("/:id", getCategoryById);
router.put("/:id", updateCategory);
router.delete("/:id", deleteCategory);

module.exports = router;
```

---

## Step 12: Create index.js

```javascript
const express = require("express");
const mongoose = require("mongoose");
const categoryRoutes = require("./routes/categoryRoutes");

const app = express();

app.use(express.json());

mongoose
  .connect("mongodb://127.0.0.1:27017/notesdb")
  .then(() => console.log("MongoDB connected"))
  .catch((err) => console.log("Connection error:", err.message));

app.use("/categories", categoryRoutes);

app.listen(5000, () => {
  console.log("Server running on port 5000");
});
```

`/categories` will be added before every route in `categoryRoutes.js`, so `router.get("/:id")` becomes `/categories/:id`.

---

## Step 13: Run the Project

```bash
npm run dev
```

You should see:

```
Server running on port 5000
MongoDB connected
```

---

