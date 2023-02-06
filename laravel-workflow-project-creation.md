# Laravel Work Flow Project Creation

1. Project creation

   ```sh
   # laravel version n<8
   composer create-project laravel/laravel <project-name>
   #  laravel version 9
   laravel new <project-name>
   ```

2. Controllers

   - working in controller folder
   - contains the function and the controller backend of the server

   ```sh
   # plain controller
   php artisan make:controller <controller-name>
   # resource controller with CRUD functions
   php artisan make:controller <controller-name> -r
   ```

3. Routing
   modifying files in route folder. ex. web.php, api.php, etc

   ```php
    # Specific Route
    Route::<method>('/<route>',[<controllerClass>::class,'<function-name>']);
    # Resource Route
    Route::resource('/<route>',<controllerClass>::class);
   ```

4. Modeling
   handling models

   ```php
   # adding fillable to the Model class you generate
   protected $fillable = [
   	'id'
   	...add field here...
   ];
   ```

5. Database Configuration and migration

   - modifying .env
   - create migration

   ```sh
   	 #ex. php artisan make:migration create_activities_table
   	 php artisan make:migration <migration-name>
   ```

6. Run Server

   running php laravel web project.

   ```sh
   php artisan serve
   ```
