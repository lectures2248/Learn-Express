# Express.js Routes and Controllers

## Step 1: Folder Structure

```
student-api/
├── controllers/
│   └── studentController.js
├── models/
│   └── Student.js
├── routes/
│   └── studentRoutes.js
├── index.js
└── package.json
```

`models/Student.js` will remain same as class 1.

---

## Step 2: Create the Controller

Open `controllers/studentController.js` and import the model:

```javascript
const Student = require("../models/Student");
```

---

## Step 3: Create Student

```javascript
const createStudent = async (req, res) => {
  try {
    const student = await Student.create(req.body);
    res.status(201).json(student);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};
```

---

## Step 4: Get All Students

```javascript
const getStudents = async (req, res) => {
  try {
    const students = await Student.find();
    res.json(students);
  } catch (err) {
    res.status(500).json({ message: err.message });
  }
};
```

---

## Step 5: Get One Student by ID

```javascript
const getStudentById = async (req, res) => {
  try {
    const student = await Student.findById(req.params.id);

    if (!student) {
      return res.status(404).json({ message: "Student not found" });
    }

    res.json(student);
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
};
```

---

## Step 6: Update Student

```javascript
const updateStudent = async (req, res) => {
  try {
    const student = await Student.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
      runValidators: true,
    });

    if (!student) {
      return res.status(404).json({ message: "Student not found" });
    }

    res.json(student);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};
```

---

## Step 7: Delete Student

```javascript
const deleteStudent = async (req, res) => {
  try {
    const student = await Student.findByIdAndDelete(req.params.id);

    if (!student) {
      return res.status(404).json({ message: "Student not found" });
    }

    res.json({ message: "Student deleted successfully" });
  } catch (err) {
    res.status(400).json({ message: "Invalid ID" });
  }
};
```

---

## Step 8: Export the Functions

At the end of `studentController.js`:

```javascript
module.exports = {
  createStudent,
  getStudents,
  getStudentById,
  updateStudent,
  deleteStudent,
};
```

---

## Step 9: Create the Routes

Open `routes/studentRoutes.js`:

```javascript
const express = require("express");
const {
  createStudent,
  getStudents,
  getStudentById,
  updateStudent,
  deleteStudent,
} = require("../controllers/studentController");

const router = express.Router();

router.post("/", createStudent);
router.get("/", getStudents);
router.get("/:id", getStudentById);
router.put("/:id", updateStudent);
router.delete("/:id", deleteStudent);

module.exports = router;
```

---

## Step 10: Update index.js

Remove the CRUD code from `index.js` and use the routes file:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const studentRoutes = require("./routes/studentRoutes");

const app = express();

app.use(express.json());

mongoose
  .connect("mongodb://127.0.0.1:27017/studentdb")
  .then(() => console.log("MongoDB connected"))
  .catch((err) => console.log("Connection error:", err.message));

app.use("/students", studentRoutes);

app.listen(5000, () => {
  console.log("Server running on port 5000");
});
```

`/students` will be added before every route in `studentRoutes.js`, so `router.get("/:id")` becomes `/students/:id`.

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
