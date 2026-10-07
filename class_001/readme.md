# Express.js CRUD Operations with Mongoose


## Step 1: Create the Project Folder

Open the terminal and run:

```bash

npm init -y
```

This creates a `package.json` file for our project.

---

## Step 2: Install the Packages

```bash
npm install express mongoose
npm install --save-dev nodemon
```

- `express` is used to create the server.
- `mongoose` is used to talk to MongoDB.
- `nodemon` restarts the server automatically whenever we save a file.

Now open `package.json` and add this inside `"scripts"`:

```json
"scripts": {
  "dev": "nodemon index.js"
}
```

---

## Step 3: Folder Structure

```
student-api/
├── models/
│   └── Student.js
├── index.js
└── package.json
```

---

## Step 4: Create the Student Model

The model tells Mongoose what a student record should look like. Open `models/Student.js`:

```javascript
const mongoose = require("mongoose");

const studentSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
    },
    course: {
      type: String,
      required: true,
    },
    age: {
      type: Number,
      min: 15,
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model("Student", studentSchema);
```

- `required: true` means this field must be sent.
- `unique: true` means two students cannot have the same email.
- `timestamps: true` adds `createdAt` and `updatedAt` fields automatically.

---

## Step 5: Create the Server and Connect to MongoDB

Make sure MongoDB is running on your machine. Then open `index.js`:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const Student = require("./models/Student");

const app = express();

// allows us to read JSON data from the request body
app.use(express.json());

// connect to MongoDB
mongoose
  .connect("mongodb://127.0.0.1:27017/studentdb")
  .then(() => console.log("MongoDB connected"))
  .catch((err) => console.log("Connection error:", err.message));

// CRUD code will go here

app.listen(5000, () => {
  console.log("Server running on port 5000");
});
```

`studentdb` is the database name. We don't need to create it ourselves, MongoDB will create it when the first record is saved.

We will now add the CRUD code one by one in place of the `// CRUD code will go here` comment.

---

## Step 6: Create (Add a New Student)

```javascript
app.post("/students", async (req, res) => {
  try {
    const student = await Student.create(req.body);
    res.status(201).json(student);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});
```

`Student.create()` takes the data from the request body and saves it in the database. Status `201` means something new was created.

---

## Step 7: Read (Get All Students)

```javascript
app.get("/students", async (req, res) => {
  try {
    const students = await Student.find();
    res.json(students);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
});
```

`Student.find()` with no condition returns every student in the collection.

---

## Step 8: Read (Get One Student by ID)

```javascript
app.get("/students/:id", async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);

    if (!student) {
      return res.status(404).json({ message: "Student not found" });
    }

    res.json(student);
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
});
```

`req.params.id` reads the ID from the URL. If no student matches, we send `404`.

---

## Step 9: Update a Student

```javascript
app.put("/products/:id", async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);

    if (!product) {
      return res.status(404).json({ message: "Product not found" });
    }

    await Product.updateOne({ _id: req.params.id }, req.body);
    res.json({ message: "Product updated" });
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
});

```

---

## Step 10: Delete a Student

```javascript
app.delete("/products/:id", async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);

    if (!product) {
      return res.status(404).json({ message: "Product not found" });
    }

    await Product.deleteOne({ _id: req.params.id });
    res.json({ message: "Product deleted" });
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
```

---

## Step 11: Run the Project

```bash
npm run dev
```

You should see:

```
Server running on port 5000
MongoDB connected
```
---
