# CodeIgniter 4 views with Blade

CodeIgniter 4 is known for its flexibility and lightweight nature, but sometimes you might crave the convenience and expressiveness of a templating engine like Blade. This guide will walk you through seamlessly integrating Blade into your CodeIgniter 4 project using the manuelgil/ci4-blade repository.

## Prerequisites:

- PHP 8.1 or later
- Composer installed

### Installation:

1. Clone the manuelgil/ci4-blade repository into your project's root directory:

```Bash
git clone https://github.com/ManuelGil/ci4-blade.git
```

2. Install dependencies:

```Bash
composer install
```

## Configuration:

Open app/Config/Paths.php and update the following paths if needed:

'templates' specifies the directory containing your Blade templates (default: Views/blades).
'layouts' (optional) defines the directory for layout files (default: templates/\_layouts).

Edit app/Config/Paths.php to adjust these paths if needed:

```php
'templates' => 'Views/blades', // Blade template directory
'layouts' => 'templates/_layouts', // Layout directory (optional)
```

## Using Blade Templates:

Create your Blade template files using the .blade.php extension (e.g., Views/blades/home.blade.php).

Use the familiar view() helper in your controllers to render the templates:

```php
class HomeController extends BaseController
{
public function index()
{
$data = [
'name' => 'John Doe',
];

        return $this->view('blades/home', $data);
    }

}
```
