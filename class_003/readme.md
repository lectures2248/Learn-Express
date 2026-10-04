# Express.js User Register API with Mongoose

## Step 1: Create the Project Folder

Open the terminal and run:

```bash
npm init -y
```

This creates a `package.json` file for our project.

---

## Step 2: Install the Packages

```bash
npm install express mongoose bcryptjs
npm install --save-dev nodemon
```

- `express` is used to create the server.
- `mongoose` is used to talk to MongoDB.
- `bcryptjs` is used to hash the password before saving it.
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
user-api/
├── controllers/
│   └── userController.js
├── models/
│   └── User.js
├── routes/
│   └── userRoutes.js
├── index.js
└── package.json
```

In class 1 we wrote everything inside `index.js`. In this project we will divide the code into three parts:

- `models` will have the structure of our data.
- `controllers` will have the actual logic, like saving the user.
- `routes` will decide which URL calls which controller function.

This way `index.js` stays small and every file has its own job. When the project gets bigger, it becomes easy to find and change the code.

---

## Step 4: Create the User Model

Open `models/User.js`:

```javascript
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
    },
    email: {
      type: String,
      required: true,
      unique: true,
      lowercase: true,
    },
    password: {
      type: String,
      required: true,
    },
  },
  { timestamps: true }
);

module.exports = mongoose.model("User", userSchema);
```

- `required: true` means this field must be sent.
- `unique: true` means two users cannot register with the same email.
- `lowercase: true` saves the email in small letters, so `Ali@Gmail.com` and `ali@gmail.com` are treated as the same email.
- `timestamps: true` adds `createdAt` and `updatedAt` fields automatically.

---

## Step 5: Create the Register Controller

Open `controllers/userController.js`:

```javascript
const User = require("../models/User");
const bcrypt = require("bcryptjs");

const registerUser = async (req, res) => {
  try {
    const { name, email, password } = req.body;

    if (!name || !email || !password) {
      return res.status(400).json({ message: "All fields are required" });
    }

    if (password.length < 6) {
      return res.status(400).json({ message: "Password must be at least 6 characters" });
    }

    const existingUser = await User.findOne({ email: email.toLowerCase() });
    if (existingUser) {
      return res.status(400).json({ message: "Email already registered" });
    }

    const hashedPassword = await bcrypt.hash(password, 10);

    const user = await User.create({
      name,
      email,
      password: hashedPassword,
    });

    res.status(201).json({
      message: "User registered successfully",
      user: {
        _id: user._id,
        name: user.name,
        email: user.email,
      },
    });
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
};

module.exports = { registerUser };
```

Here is what happens in order:

1. We take `name`, `email` and `password` from `req.body`.
2. If any field is empty, we stop and send an error.
3. If the password is less than 6 characters, we stop. We check this here and not in the model, because the model will receive the hashed password, which is always long.
4. We check if a user with this email already exists. If yes, we stop.
5. `bcrypt.hash(password, 10)` converts the password into a long hashed string. The number `10` is the salt rounds, higher number means stronger hash but slower.
6. We save the user with the hashed password, not the real one.
7. In the response we only send `_id`, `name` and `email`. We never send the password back, even if it is hashed.

Status `201` means something new was created. Status `400` means there is a problem in the data sent by the user.

If you open MongoDB Compass after registering, you will see the password saved like `$2a$10$Xk9...`. Even the admin of the database cannot read the real password.

---

## Step 6: Create the Route

Open `routes/userRoutes.js`:

```javascript
const express = require("express");
const { registerUser } = require("../controllers/userController");

const router = express.Router();

router.post("/register", registerUser);

module.exports = router;
```

`express.Router()` works like a small `app`. We use `router.post()` the same way we used `app.post()` in class 1. The only difference is that the function is written in the controller file and we just pass its name here.

We use `POST` because we are sending new data to the server. The data goes in the body, not in the URL, so the password is not visible in the URL.

---

## Step 7: Create index.js

Open `index.js`:

```javascript
const express = require("express");
const mongoose = require("mongoose");
const userRoutes = require("./routes/userRoutes");

const app = express();

app.use(express.json());

mongoose
  .connect("mongodb://127.0.0.1:27017/userdb")
  .then(() => console.log("MongoDB connected"))
  .catch((err) => console.log("Connection error:", err.message));

app.use("/users", userRoutes);

app.listen(5000, () => {
  console.log("Server running on port 5000");
});
```

`app.use(express.json())` allows us to read JSON data from `req.body`. Without it, `req.body` will be `undefined`.

`app.use("/users", userRoutes)` adds `/users` before the route in `userRoutes.js`. So `router.post("/register")` becomes `/users/register`.

---

## Step 8: Run the Project

```bash
npm run dev
```

You should see:

```
Server running on port 5000
MongoDB connected
```

---

## Step 9: Test in Postman

Select `POST`, enter the URL, then go to Body, select raw and JSON.

`POST http://localhost:5000/users/register`

```json
{
  "name": "Ali Raza",
  "email": "ali@example.com",
  "password": "ali12345"
}
```

Response:

```json
{
  "message": "User registered successfully",
  "user": {
    "_id": "66ff1a2b3c4d5e6f7a8b9c0d",
    "name": "Ali Raza",
    "email": "ali@example.com"
  }
}
```
