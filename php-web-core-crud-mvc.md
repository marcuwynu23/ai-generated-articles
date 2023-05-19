# PHP Web Core CRUD Implementation with MVC Architecture

## Introduction

In web development, CRUD operations (Create, Read, Update, Delete) are fundamental for interacting with a database. In this article, we will explore how to implement CRUD operations in a PHP web application using the MVC (Model-View-Controller) architectural pattern. The MVC pattern separates concerns and provides a structured approach to building scalable and maintainable applications. We'll cover the basics of MVC and demonstrate how to implement CRUD functionality using PHP's core features.

## Prerequisites

Before we begin, ensure that you have the following installed:

- PHP (version 7 or above)
- A web server (e.g., Apache, Nginx)
- A database system (e.g., MySQL, PostgreSQL)

## Step 1: Setting Up the Project

Start by creating a new directory for your PHP web application. Open a text editor and navigate to the project directory. Create the following directories: models, views, and controllers. These directories will hold our PHP files according to the MVC pattern.

## Step 2: Database Integration

To interact with the database, we'll use PHP's PDO (PHP Data Objects) extension. Create a new PHP file in the models directory, let's call it Database.php. This file will handle the database connection and provide methods for executing queries. Here's an example:

```php
<?php

class Database
{
    private $host = 'localhost';
    private $username = 'your_username';
    private $password = 'your_password';
    private $database = 'your_database';

    private $connection;

    public function __construct()
    {
        $dsn = "mysql:host={$this->host};dbname={$this->database};charset=utf8mb4";
        $this->connection = new PDO($dsn, $this->username, $this->password);
        $this->connection->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    }

    public function executeQuery($query, $params = [])
    {
        $stmt = $this->connection->prepare($query);
        $stmt->execute($params);
        return $stmt;
    }
}
```

## Step 3: Model Implementation

Models represent the data structures and logic of your application. Create a new PHP file in the models directory, let's call it TodoModel.php. This file will handle CRUD operations for a hypothetical "Todo" entity. Here's an example implementation:

```php
<?php

require_once 'Database.php';

class TodoModel
{
    private $db;

    public function __construct()
    {
        $this->db = new Database();
    }

    public function create($data)
    {
        $query = "INSERT INTO todos (title, description) VALUES (?, ?)";
        $params = [$data['title'], $data['description']];
        $this->db->executeQuery($query, $params);
        // Handle success or error response
    }

    public function read($id)
    {
        $query = "SELECT * FROM todos WHERE id = ?";
        $params = [$id];
        $stmt = $this->db->executeQuery($query, $params);
        $todo = $stmt->fetch(PDO::FETCH_ASSOC);
        // Return the retrieved todo or handle not found case
    }

    public function update($id, $data)
    {
        $query = "UPDATE todos SET title = ?, description = ? WHERE id = ?";
        $params = [$data['title'], $data['description'], $id];
        $this->db->executeQuery($query, $params);
        // Handle success or error response
    }

    public function delete($id)
    {
        $query = "DELETE FROM todos WHERE id = ?";
        $params = [$id];
        $this->db->executeQuery($query, $params);
        // Handle success or error response
    }
}
```

## Step 4: Controller Implementation

Controllers handle the application logic and orchestrate interactions between models and views. Create a new PHP file in the controllers directory, let's call it TodoController.php. This file will handle user requests and invoke the corresponding model methods. Here's an example implementation:

```php
<?php

require_once 'models/TodoModel.php';

class TodoController
{
    private $model;

    public function __construct()
    {
        $this->model = new TodoModel();
    }

    public function create()
    {
        // Retrieve and validate request data

        // Invoke the model's create method
        $this->model->create($data);
    }

    public function read($id)
    {
        // Invoke the model's read method
        $todo = $this->model->read($id);
        // Render the view or return JSON response
    }

    public function update($id)
    {
        // Retrieve and validate request data

        // Invoke the model's update method
        $this->model->update($id, $data);
    }

    public function delete($id)
    {
        // Invoke the model's delete method
        $this->model->delete($id);
    }
}
```

## Step 5: View Implementation

Views handle the presentation logic of your application. Create a new PHP file in the views directory, let's call it todo_view.php. This file will display the data retrieved from the controller. Here's an example implementation:

```php
<?php

class TodoView
{
    public function render($todo)
    {
        // Render the todo details
        echo "<h1>{$todo['title']}</h1>";
        echo "<p>{$todo['description']}</p>";
    }
}
```

## Step 6: Routing and User Interface

To complete the implementation, you'll need to define routes that map user requests to the corresponding controller methods. Additionally, you'll need to create a user interface (UI) for interacting with the CRUD functionality, such as HTML forms for creating and updating records, and appropriate links or buttons for deleting records.

## Conclusion

By following the MVC pattern, you can build a structured and maintainable PHP web application with CRUD functionality. Models handle database interactions, controllers manage the application logic, and views handle the presentation of data. Remember to handle user input validation, error handling, and implement proper security measures to ensure the integrity of your application. Happy coding!

Note: This article demonstrates a simplified implementation of MVC in PHP. In real-world scenarios, you might use frameworks like Laravel, Symfony, or CodeIgniter that provide MVC out-of-the-box, along with additional features and best practices.
