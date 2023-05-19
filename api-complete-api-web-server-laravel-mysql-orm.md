# How to Create a Complete API Web Server in Laravel with MySQL ORM

Creating a complete API web server in Laravel with MySQL ORM can seem like a daunting task, but with the right approach, it can be easy and straightforward. In this article, we'll walk through the steps to create a complete API web server using Laravel and the MySQL ORM, with code examples.


## Step 1: Setting up the environment

First, we need to set up our development environment. We'll assume that you have PHP, Composer, and MySQL installed on your system. To create a new Laravel project, run the following command:

``` sh
composer create-project --prefer-dist laravel/laravel api-app
```

## Step 2: Configuring the database

Next, we need to configure the database. Open the .env file and update the database configuration to match your local environment. For example:

```makefile
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=api_db
DB_USERNAME=root
DB_PASSWORD=
```

## Step 3: Creating the API routes
We'll create the API routes in the routes/api.php file. For example:

```php
Route::get('/users', 'UserController@index');
Route::get('/users/{id}', 'UserController@show');
Route::post('/users', 'UserController@store');
Route::put('/users/{id}', 'UserController@update');
Route::delete('/users/{id}', 'UserController@destroy');
```

## Step 4: Creating the controller

We'll create the UserController controller using the following command:

```sh
php artisan make:controller UserController --api
```

In the UserController controller, we'll add the methods to handle the API requests:

```php
class UserController extends Controller
{
public function index()
{
$users = User::all();
return response()->json($users);
}

public function show($id)
{
    $user = User::find($id);
    return response()->json($user);
}

public function store(Request $request)
{
    $user = new User;
    $user->name = $request->input('name');
    $user->email = $request->input('email');
    $user->save();
    return response()->json($user);
}

public function update(Request $request, $id)
{
    $user = User::find($id);
    $user->name = $request->input('name');
    $user->email = $request->input('email');
    $user->save();
    return response()->json($user);
}

public function destroy($id)
{
    $user = User::find($id);
    $user->delete();
    return response()->json(['message' => 'User deleted']);
}
}
```
## Step 5: Testing the API

Finally, we can test the API using a tool like Postman. We can make requests to the API routes and receive JSON responses.

In conclusion, creating a complete API web server in Laravel with MySQL ORM is a simple and powerful way to build web applications. By following the steps outlined above and using the code examples provided, you'll be able to create a robust and scalable API in no time.