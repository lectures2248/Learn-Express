# React Register Page with Axios

## Step 1: Enable CORS in the Backend

Open the `user-api` project from class 2 and install `cors`:

```bash
npm install cors
```

Open `index.js` and add:

```javascript
const cors = require("cors");
```

```javascript
app.use(cors({ origin: "http://localhost:5173" }));
app.use(express.json());
```

React runs on port `5173` and the backend on port `5000`. Without `cors`, the browser will block the request.

Keep the backend running with `npm run dev`.

---

## Step 2: Create the React App

In a new terminal, outside the `user-api` folder:

```bash
npm create vite@latest register-app -- --template react
cd register-app
npm install
npm install axios formik
```

Remove everything inside `src/index.css` and `src/App.css`.

---

## Step 3: Folder Structure

```
register-app/
├── src/
│   ├── api/
│   │   └── axios.js
│   ├── services/
│   │   └── userService.js
│   ├── validations/
│   │   └── registerValidation.js
│   ├── hooks/
│   │   └── useRegister.js
│   ├── components/
│   │   ├── InputField.jsx
│   │   └── Message.jsx
│   ├── pages/
│   │   ├── Register.jsx
│   │   └── Register.css
│   ├── App.jsx
│   └── main.jsx
├── .env
└── package.json
```

- `api` has the axios setup.
- `services` has the API calls.
- `validations` has the form rules.
- `hooks` has the form logic, Formik and the API call together.
- `components` has small UI parts that we can use again in other forms.
- `pages` has only the screen, no logic.

---

## Step 4: Add the Backend URL in .env

Create `.env` next to `package.json`:

```
VITE_API_URL=http://localhost:5000
```

In Vite, the variable name must start with `VITE_`. Restart the React app after changing `.env`.

---

## Step 5: Create the Axios Instance

Open `src/api/axios.js`:

```javascript
import axios from "axios";

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  headers: {
    "Content-Type": "application/json",
  },
});

export default api;
```

`baseURL` is added before every request, so `api.post("/users/register")` goes to `http://localhost:5000/users/register`.

---

## Step 6: Create the User Service

Open `src/services/userService.js`:

```javascript
import api from "../api/axios";

export const registerUser = async (userData) => {
  const response = await api.post("/users/register", userData);
  return response.data;
};
```

This sends the same request we sent from Postman in class 2. The backend response is inside `response.data`.

---

## Step 7: Create the Validation

Open `src/validations/registerValidation.js`:

```javascript
export const registerValidation = (values) => {
  const errors = {};

  if (!values.name) {
    errors.name = "Name is required";
  }

  if (!values.email) {
    errors.email = "Email is required";
  } else if (!/^\S+@\S+\.\S+$/.test(values.email)) {
    errors.email = "Enter a valid email";
  }

  if (!values.password) {
    errors.password = "Password is required";
  } else if (values.password.length < 6) {
    errors.password = "Password must be at least 6 characters";
  }

  return errors;
};
```

Formik calls this function with the form values. If the `errors` object is empty, the form will be submitted.

---

## Step 8: Create the useRegister Hook

Open `src/hooks/useRegister.js`:

```javascript
import { useState } from "react";
import { useFormik } from "formik";
import { registerUser } from "../services/userService";
import { registerValidation } from "../validations/registerValidation";

export const useRegister = () => {
  const [error, setError] = useState("");
  const [success, setSuccess] = useState("");

  const formik = useFormik({
    initialValues: {
      name: "",
      email: "",
      password: "",
    },
    validate: registerValidation,
    onSubmit: async (values, { resetForm }) => {
      setError("");
      setSuccess("");

      try {
        const data = await registerUser(values);
        setSuccess(data.message);
        resetForm();
      } catch (err) {
        if (err.response) {
          setError(err.response.data.message);
        } else {
          setError("Cannot connect to server");
        }
      }
    },
  });

  return { formik, error, success };
};
```

All the logic of the register form is in this hook. The page will just call `useRegister()` and use what it returns.

`onSubmit` runs only when validation passes. `resetForm()` clears the form after register.

`err.response` has the error sent by our backend, like `Email already registered`. If the server is off, there is no `err.response`, so we show our own message.

---

## Step 9: Create the InputField Component

Open `src/components/InputField.jsx`:

```jsx
function InputField({ label, name, type = "text", formik }) {
  return (
    <div className="form-group">
      <label htmlFor={name}>{label}</label>
      <input id={name} type={type} {...formik.getFieldProps(name)} />
      {formik.touched[name] && formik.errors[name] && (
        <span className="field-error">{formik.errors[name]}</span>
      )}
    </div>
  );
}

export default InputField;
```

Label, input and error message are in one component, so we do not repeat the same code for every field.

`formik.getFieldProps(name)` gives `name`, `value`, `onChange` and `onBlur` to the input.

`formik.touched[name]` becomes true when the user leaves the input, so the error does not show before the user has typed anything.

---

## Step 10: Create the Message Component

Open `src/components/Message.jsx`:

```jsx
function Message({ type, text }) {
  if (!text) return null;

  return <p className={`message ${type}`}>{text}</p>;
}

export default Message;
```

If there is no text, nothing is shown.

---

## Step 11: Create the Register Page

Open `src/pages/Register.jsx`:

```jsx
import InputField from "../components/InputField";
import Message from "../components/Message";
import { useRegister } from "../hooks/useRegister";
import "./Register.css";

function Register() {
  const { formik, error, success } = useRegister();

  return (
    <div className="register-container">
      <form className="register-form" onSubmit={formik.handleSubmit}>
        <h2>Create Account</h2>

        <InputField label="Name" name="name" formik={formik} />
        <InputField label="Email" name="email" type="email" formik={formik} />
        <InputField label="Password" name="password" type="password" formik={formik} />

        <button type="submit" disabled={formik.isSubmitting}>
          {formik.isSubmitting ? "Registering..." : "Register"}
        </button>

        <Message type="error" text={error} />
        <Message type="success" text={success} />
      </form>
    </div>
  );
}

export default Register;
```

The page has only UI now. All the logic comes from `useRegister()`.

`formik.isSubmitting` is true while the request is going, Formik handles it on its own.

---

## Step 12: Add the Styles

Open `src/pages/Register.css`:

```css
.register-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f3f4f6;
  font-family: Arial, sans-serif;
}

.register-form {
  width: 100%;
  max-width: 380px;
  background: #ffffff;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.register-form h2 {
  margin: 0 0 20px;
  text-align: center;
}

.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 15px;
}

.form-group label {
  font-size: 14px;
  margin-bottom: 5px;
}

.form-group input {
  padding: 10px;
  border: 1px solid #d1d5db;
  border-radius: 5px;
}

.field-error {
  color: #b91c1c;
  font-size: 13px;
  margin-top: 5px;
}

.register-form button {
  width: 100%;
  padding: 11px;
  margin-top: 5px;
  background: #2563eb;
  color: #ffffff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.register-form button:disabled {
  background: #93c5fd;
  cursor: not-allowed;
}

.message {
  margin: 15px 0 0;
  padding: 10px;
  border-radius: 5px;
  text-align: center;
}

.error {
  background: #fee2e2;
  color: #b91c1c;
}

.success {
  background: #dcfce7;
  color: #15803d;
}
```

---

## Step 13: Update App.jsx

Open `src/App.jsx`:

```jsx
import Register from "./pages/Register";

function App() {
  return <Register />;
}

export default App;
```

---

## Step 14: Run the App

```bash
npm run dev
```

Open `http://localhost:5173` in the browser and register a user. Try the same email again to see the error from the backend.

---
