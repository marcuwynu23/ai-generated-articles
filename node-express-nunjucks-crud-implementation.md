# Creating a CRUD App with Express.js and Nunjucks

## Step 1: Setup

To start with, we need to set up our project. Follow these steps:

1. Create a new directory for your project and navigate to it in your terminal.

2. Run the following command to create a new package.json file:

   ```sh
   npm init -y
   ```

3. Install the necessary dependencies:
   ```sh
   npm install express nunjucks body-parser --save
   ```
4. Create a new file called server.js and add the following code to it:

   ```js
   const express = require("express");
   const nunjucks = require("nunjucks");
   const bodyParser = require("body-parser");
   const app = express();

   nunjucks.configure("views", {
     autoescape: true,
     express: app,
   });

   // Configure body-parser
   app.use(bodyParser.urlencoded({ extended: false }));

   // Start the server
   const PORT = process.env.PORT || 3000;
   app.listen(PORT, () => {
     console.log(`Server listening on port ${PORT}`);
   });
   ```

   This code does the following:

   - Imports the necessary dependencies (express, nunjucks, and body-parser).
   - Creates a new instance of the express application.
   - Configures Nunjucks to use the views directory for storing view templates.
   - Configures body-parser to parse incoming request bodies as x-www-form-urlencoded.
   - Starts the server on port 3000.
   - Create a new directory called models. This directory will contain our data model.

## Step 2: Creating the Model

Next, we need to create a simple data model to represent our users. Create a new file called user.js in the models directory and add the following code to it:

```js
let users = [
  {
    id: 1,
    name: "John Doe",
    email: "john.doe@example.com",
  },
  {
    id: 2,
    name: "Jane Doe",
    email: "jane.doe@example.com",
  },
];

class User {
  static all() {
    return users;
  }

  static find(id) {
    return users.find((user) => user.id === id);
  }

  static create({ name, email }) {
    const user = { id: users.length + 1, name, email };
    users.push(user);
    return user;
  }

  static update(id, { name, email }) {
    const index = users.findIndex((user) => user.id === id);
    if (index === -1) {
      return null;
    }
    const user = { ...users[index], name, email };
    users[index] = user;
    return user;
  }

  static delete(id) {
    const index = users.findIndex((user) => user.id === id);
    if (index === -1) {
      return null;
    }
    const user = users[index];
    users.splice(index, 1);
    return user;
  }
}

module.exports = User;
```

This code defines a User class with the following static methods:

- all() - returns an array of all users.
- find(id) - returns the user with the specified id.
- create({ name, email }) - creates a new user with the specified name and email.
- update(id, { name, email }) - updates the user with the specified id and returns the updated user.
- delete(id) - deletes the user with the specified id and returns the deleted user.

The users array is used to store our user data.

## Step 3: Creating the Views

Next, we need to create the views that our users will see. Create a new directory called views. Inside this directory, create the following files:

- index.njk - the view that displays a list of all users.
- new.njk - the view that displays a form for creating a new user.
- edit.njk - the view that displays a form for editing an existing user.
- show.njk - the view that displays the details of a specific user.
- delete.njk - the view that displays a confirmation message for deleting a user.

Add the following code to each of these files:

### index.njk

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Users</title>
  </head>
  <body>
    <h1>Users</h1>
    <ul>
      {% for user in users %}
      <li>
        <a href="/users/{{ user.id }}">{{ user.name }}</a>
      </li>
      {% endfor %}
    </ul>
    <a href="/users/new">New User</a>
  </body>
</html>
```

In this view, we have a heading that says "Users", followed by a list of all users. Each user is displayed as a link with their name, which will take the user to the show view for that user. We also have a link to the new view for creating a new user.

### new.njk

```html
<!DOCTYPE html>
<html>
  <head>
    <title>New User</title>
  </head>
  <body>
    <h1>New User</h1>
    <form action="/users" method="POST">
      <div>
        <label for="name">Name:</label>
        <input type="text" name="name" />
      </div>
      <div>
        <label for="email">Email:</label>
        <input type="email" name="email" />
      </div>
      <button type="submit">Create</button>
    </form>
  </body>
</html>
```

In this view, we have a form for creating a new user. The form has two input fields, one for the user's name and one for their email address. We have also used a POST method to submit the form data to the server.

### edit.njk

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Edit User</title>
  </head>
  <body>
    <h1>Edit User</h1>
    <form action="/users/{{ user.id }}" method="POST">
      <input type="hidden" name="_method" value="PUT" />
      <div>
        <label for="name">Name:</label>
        <input type="text" name="name" value="{{ user.name }}" />
      </div>
      <div>
        <label for="email">Email:</label>
        <input type="email" name="email" value="{{ user.email }}" />
      </div>
      <button type="submit">Update</button>
    </form>
  </body>
</html>
```

In this view, we have a form for editing an existing user. The form has two input fields, one for the user's name and one for their email address. We have also used a POST method to submit the form data to the server, along with a hidden input field named \_method with a value of "PUT". This tells Express to treat the request as an update request.

### show.njk

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{{ user.name }}</title>
  </head>
  <body>
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
    <a href="/users/{{ user.id }}/edit">Edit</a>
    <form action="/users/{{ user.id }}" method="POST">
      <input type="hidden" name="_method" value="DELETE" />
      <button type="submit">Delete</button>
    </form>
  </body>
</html>
```

In this view, we have the details of a specific user. We have displayed the user's name and email address, as well as links to edit and delete the user. We have used a DELETE method for the delete form, which tells Express to treat the request as a delete request.

### delete.njk

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Delete User</title>
  </head>
  <body>
    <h1>Delete User</h1>
    <p>Are you sure you want to delete this user?</p>
    <form action="/users/{{ user.id }}" method="POST">
      <input type="hidden" name="_method" value="DELETE" />
      <button type="submit">Yes</button>
      <a href="/users/{{ user.id }}">No</a>
    </form>
  </body>
</html>
```

In this view, we have a confirmation message for deleting a user. We have used a DELETE method for the delete form, which tells Express to treat the request as a delete request. We have also included a "No" link for the user to cancel the delete request and return to the show view.

## Step 4: Creating the Routes

Next, we need to create the routes that will handle the HTTP requests from our views. Create a new file called routes.js and add the following code:

```js
const express = require("express");
const router = express.Router();
const { v4: uuidv4 } = require("uuid");

let users = [];

// Index Route
router.get("/", (req, res) => {
  res.render("index.njk", { users });
});

// New Route
router.get("/new", (req, res) => {
  res.render("new.njk");
});

router.post("/", (req, res) => {
  const { name, email } = req.body;
  const id = uuidv4();
  const user = { id, name, email };
  users.push(user);
  res.redirect("/");
});

// Edit Route
router.get("/:id/edit", (req, res) => {
  const user = users.find((user) => user.id === req.params.id);
  res.render("edit.njk", { user });
});

router.put("/:id", (req, res) => {
  const { name, email } = req.body;
  const user = users.find((user) => user.id === req.params.id);
  user.name = name;
  user.email = email;
  res.redirect(`/users/${req.params.id}`);
});

// Show Route
router.get("/:id", (req, res) => {
  const user = users.find((user) => user.id === req.params.id);
  res.render("show.njk", { user });
});

// Delete Route
router.get("/:id/delete", (req, res) => {
  const user = users.find((user) => user.id === req.params.id);
  res.render("delete.njk", { user });
});

router.delete("/:id", (req, res) => {
  users = users.filter((user) => user.id !== req.params.id);
  res.redirect("/");
});

module.exports = router;
```

Here, we have created a new router using `express.Router()`. We have defined routes for the `index`, `new`, `edit`, `show`, and `delete` views. We have used the `uuid` module to generate unique IDs for each new user.

For the `POST` route, we extract the `name` and `email` values from the request body, generate a new ID, create a new user object, and push it onto the `users` array. Then we redirect the user to the `index` view.

For the `PUT` route, we extract the `name` and `email` values from the request body, find the user in the `users` array with the specified ID, and update their name and email. Then we redirect the user to the `show` view.

For the `DELETE` route, we remove the user with the specified ID from the `users` array and redirect the user to the `index` view.

## Step 5: Setting up the App

Finally, we need to set up the app to use our routes and views. Create a new file called `app.js` and add the following code:

```js
const express = require("express");
const app = express();
const nunjucks = require("nunjucks");
const routes = require("./routes");

// Set the view engine to use Nunjucks
nunjucks.configure("views", {
  autoescape: true,
  express: app,
});

// Parse request bodies as JSON
app.use(express.json());

// Parse request bodies as URL encoded data
app.use(express.urlencoded({ extended: true }));

// Set up our routes
app.use("/users", routes);

// Start the server
const port = process.env.PORT || 3000;
app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
```

Here, we have created a new express app and set up nunjucks as our view engine. We have configured the app to parse request bodies as JSON and URL-encoded data.

We have set up our routes to be under the /users prefix and mounted the routes router to handle those requests.

Finally, we start the server and listen on either the PORT environment variable or port 3000.

## Step 6: Testing the App

To test the app, run node app.js in the terminal and go to http://localhost:3000 in your browser. You should see a list of users (which will be empty initially).

To add a new user, click the "New User" button and fill out the form. Then click the "Create User" button to submit the form. You should be redirected back to the index page and see the new user in the list.

To edit a user, click the "Edit" link next to their name in the index page. Fill out the form and click the "Update User" button. You should be redirected to the user's show page and see the updated information.

To delete a user, click the "Delete" link next to their name in the index page. You will be asked to confirm the deletion. Click the "Yes" button to delete the user and be redirected back to the index page.

## Conclusion

In this article, we have learned how to perform CRUD operations in Express.js using Nunjucks as our view engine. We have created views for listing, creating, editing, showing
