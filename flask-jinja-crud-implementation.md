# Creating a CRUD App with Flask and Jinja

Flask is a micro web framework written in Python that provides a simple and flexible way to create web applications. In this article, we will learn how to perform CRUD operations using Flask and Jinja, a template engine for Python.

## Prerequisites

Before we get started, make sure you have the following installed on your system:

- **Python 3.x**
- **Flask**
- **Jinja**

## Step 1: Creating a Database

First, let's create a simple database to store our data. In this example, we will be using SQLite. Create a new file called database.db and run the following SQL commands to create a users table:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    email TEXT NOT NULL
);
```

## Step 2: Creating the Flask App

Next, let's create the Flask app. Create a new file called app.py and add the following code:

```py
from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)
db = 'database.db'

@app.route('/')
def index():
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute('SELECT * FROM users')
    users = c.fetchall()
    conn.close()
    return render_template('index.html', users=users)

@app.route('/new', methods=['GET', 'POST'])
def new():
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        conn = sqlite3.connect(db)
        c = conn.cursor()
        c.execute('INSERT INTO users (name, email) VALUES (?, ?)', (name, email))
        conn.commit()
        conn.close()
        return redirect(url_for('index'))
    else:
        return render_template('new.html')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit(id):
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute('SELECT * FROM users WHERE id = ?', (id,))
    user = c.fetchone()
    conn.close()
    if request.method == 'POST':
        name = request.form['name']
        email = request.form['email']
        conn = sqlite3.connect(db)
        c = conn.cursor()
        c.execute('UPDATE users SET name = ?, email = ? WHERE id = ?', (name, email, id))
        conn.commit()
        conn.close()
        return redirect(url_for('show', id=id))
    else:
        return render_template('edit.html', user=user)

@app.route('/show/<int:id>')
def show(id):
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute('SELECT * FROM users WHERE id = ?', (id,))
    user = c.fetchone()
    conn.close()
    return render_template('show.html', user=user)

@app.route('/delete/<int:id>')
def delete(id):
    conn = sqlite3.connect(db)
    c = conn.cursor()
    c.execute('DELETE FROM users WHERE id = ?', (id,))
    conn.commit()
    conn.close()
    return redirect(url_for('index'))

if __name__ == '__main__':
    app.run(debug=True)
```

Here, we have created a new Flask app and defined five routes:

- /: This route displays a list of all the users in the database.
- /new: This route displays a form to create a new user. When the form is submitted, a new user is created in the database and the user is redirected to the home page.
- /edit/[int:id](int:id): This route displays a form to edit an existing user. When the form is submitted, the user's details are updated in the database and the user is redirected to the show page for that user.
- /show/[int:id](int:id): This route displays the details of a single user.
- /delete/[int:id](int:id): This route deletes a user from the database and redirects to the home page.
  We have also defined a db variable which stores the path to the database file.

## Step 3: Creating the Templates

Next, let's create the templates for our app. Create a new directory called templates and add the following four HTML files:

### index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>User List</title>
  </head>
  <body>
    <h1>User List</h1>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Name</th>
          <th>Email</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        {% for user in users %}
        <tr>
          <td>{{ user[0] }}</td>
          <td>{{ user[1] }}</td>
          <td>{{ user[2] }}</td>
          <td>
            <a href="{{ url_for('show', id=user[0]) }}">Show</a>
            <a href="{{ url_for('edit', id=user[0]) }}">Edit</a>
            <a href="{{ url_for('delete', id=user[0]) }}">Delete</a>
          </td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
    <a href="{{ url_for('new') }}">Create New User</a>
  </body>
</html>
```

### new.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Create New User</title>
  </head>
  <body>
    <h1>Create New User</h1>
    <form action="{{ url_for('new') }}" method="post">
      <label>Name:</label>
      <input type="text" name="name" /><br />
      <label>Email:</label>
      <input type="email" name="email" /><br />
      <input type="submit" value="Submit" />
    </form>
  </body>
</html>
```

### edit.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Edit User</title>
  </head>
  <body>
    <h1>Edit User</h1>
    <form action="{{ url_for('edit', id=user[0]) }}" method="post">
      <label>Name:</label>
      <input type="text" name="name" value="{{ user[1] }}" /><br />
      <label>Email:</label>
      <input type="email" name="email" value="{{ user[2] }}" /><br />
      <input type="submit" value="Submit" />
    </form>
  </body>
</html>
```

### show.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>User Details</title>
  </head>
  <body>
    <h1>User Details</h1>
    <table>
      <tr>
        <td>ID:</td>
        <td>{{ user[0] }}</td>
      </tr>
      <tr>
        <td>Name:</td>
        <td>{{ user[1] }}</td>
      </tr>
      <tr>
        <td>Email:</td>
        <td>{{ user[2] }}</td>
      </tr>
    </table>
    <a href="{{ url_for('edit', id=user[0]) }}">Edit</a>
    <a href="{{ url_for('delete', id=user[0]) }}">Delete</a>
  </body>
</html>
```

In these templates, we use the Jinja2 templating engine to display data from our Flask app. Note that we use the url_for function to generate URLs for our routes.

## Step 4: Implementing the Routes

Now that we have defined our routes and templates, let's implement the routes in our Flask app. Open the app.py file and replace its contents with the following:

```py
from flask import Flask, render_template, request, redirect, url_for
import sqlite3

app = Flask(__name__)

# Define the database
db = 'users.db'

# Define the routes
@app.route('/')
def index():
    # Connect to the database
    conn = sqlite3.connect(db)
    c = conn.cursor()

    # Get all the users
    c.execute('SELECT * FROM users')
    users = c.fetchall()

    # Close the database connection
    conn.close()

    # Render the template
    return render_template('index.html', users=users)

@app.route('/new', methods=['GET', 'POST'])
def new():
    if request.method == 'POST':
        # Get the form data
        name = request.form['name']
        email = request.form['email']

        # Connect to the database
        conn = sqlite3.connect(db)
        c = conn.cursor()

        # Add the user to the database
        c.execute('INSERT INTO users (name, email) VALUES (?, ?)', (name, email))
        conn.commit()

        # Close the database connection
        conn.close()

        # Redirect to the home page
        return redirect(url_for('index'))
    else:
        # Render the template
        return render_template('new.html')

@app.route('/edit/<int:id>', methods=['GET', 'POST'])
def edit(id):
    # Connect to the database
    conn = sqlite3.connect(db)
    c = conn.cursor()

    if request.method == 'POST':
        # Get the form data
        name = request.form['name']
        email = request.form['email']

        # Update the user in the database
        c.execute('UPDATE users SET name=?, email=? WHERE id=?', (name, email, id))
        conn.commit()

        # Close the database connection
        conn.close()

        # Redirect to the show page for the user
        return redirect(url_for('show', id=id))
    else:
        # Get the user from the database
        c.execute('SELECT * FROM users WHERE id=?', (id,))
        user = c.fetchone()

        # Close the database connection
        conn.close()

        # Render the template
        return render_template('edit.html', user=user)

@app.route('/show/<int:id>')
def show(id):
    # Connect to the database
    conn = sqlite3.connect(db)
    c = conn.cursor()

    # Get the user from the database
    c.execute('SELECT * FROM users WHERE id=?', (id,))
    user = c.fetchone()

    # Close the database connection
    conn.close()

    # Render the template
    return render_template('show.html', user=user)
@app.route('/delete/<int:id>')
def delete(id):
    # Connect to the database
    conn = sqlite3.connect(db)
    c = conn.cursor()

    # Delete the user from the database
    c.execute('DELETE FROM users WHERE id=?', (id,))
    conn.commit()

    # Close the database connection
    conn.close()

    # Redirect to the home page
    return redirect(url_for('index'))
```

This route handles the deletion of a user from the database. It first connects to the database, then uses a DELETE statement to remove the user with the specified id from the users table. It then commits the transaction and closes the database connection. Finally, it redirects the user to the home page.

# Step 4: Running the App

Now that we have created the Flask app and the templates, let's run the app. Open a terminal and navigate to the directory where the app.py file is located. Then, run the following command:

```py
python app.py
```

This will start the Flask development server. You should see output similar to the following:

```
 * Serving Flask app "app" (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 123-456-789
```

Open a web browser and navigate to http://127.0.0.1:5000/. You should see a list of all the users in the database. You can click on the "Create New User" link to add a new user, or click on the "Edit" or "Delete" links to modify or remove an existing user.

Congratulations, you have created a simple CRUD app using Flask and Jinja!
