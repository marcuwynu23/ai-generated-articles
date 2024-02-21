# Building a PHP Web Application with CodeIgniter

CodeIgniter is a powerful PHP framework known for its simplicity and speed. It follows the Model-View-Controller (MVC) architecture, making it easy to organize and maintain your codebase. In this article, we'll walk through the process of creating a simple web application using CodeIgniter.

## Step 1: Setting Up CodeIgniter

1. **Download CodeIgniter**: Visit the [CodeIgniter website](https://codeigniter.com/) and download the latest version of CodeIgniter.
2. **Extract Files**: Extract the downloaded file to your desired directory.
3. **Configuration**: Adjust configuration settings in `application/config/config.php` if necessary.
4. **Test Installation**: Visit your project's URL to ensure CodeIgniter is set up correctly.

or

1. **create project using composer**:

```sh
composer create-project codeigniter4/appstarter projectName
```

## Step 2: Creating a Controller

1. **Create a Controller File**: Inside the `application/controllers` directory, create a new PHP file, e.g., `Welcome.php`.
2. **Define Class**: Define a class with the same name as the file, extending `CI_Controller`.
3. **Create Methods**: Define methods within the class to handle different requests.

```php
<?php
class Welcome extends CI_Controller {
    public function index() {
        $this->load->view('welcome_message');
    }
}
```

## Step 3: Creating Views

1. Create View Files: Inside the application/views directory, create PHP files for your views, e.g., welcome_message.php.
2. Write HTML: Write HTML and PHP code in the view files to display the content.

```php
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to My Website</title>
</head>
<body>
    <h1>Welcome to My Website</h1>
    <p>This is a simple CodeIgniter MVC application.</p>
</body>
</html>
```

## Step 4: Creating a Route

1. Define Routes: Open application/config/routes.php.
2. Set Default Route: Set a default route to your controller.

```php
$route['default_controller'] = 'welcome';
```

## Step 5: Testing Your Application

Visit your project's URL in a web browser. You should see the content from your view file being displayed.

## Step 6: Expand and Refine

Continue expanding your application by adding more controllers, views, and models as needed. Configure database settings in application/config/database.php and use models to interact with the database.
