# Customizing and Using the Logger in PHP Laravel

The logging functionality in Laravel allows you to record important information, errors, and debug messages. Laravel utilizes the powerful Monolog library for logging, which provides various handlers and formatters to customize the log output.

In this article, we will explore how to customize the logger and use it effectively in a Laravel application. Additionally, we'll learn how to view real-time log updates using the `watch` and `cat` Unix commands.

## Customizing the Logger

By default, Laravel logs messages to the `storage/logs/laravel.log` file. However, you can customize the log channel and format to suit your needs.

### Changing the Log Channel

To change the log channel, open the `config/logging.php` file. In the `'channels'` array, you'll find various predefined log channels such as `'single'`, `'daily'`, and `'stack'`. Choose the appropriate channel or create a new one.

For example, to use the `'daily'` channel, modify the configuration as follows:

```php
'daily' => [
    'driver' => 'daily',
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    // ...
],
```

### Modifying the Log Format

To modify the log format, you can use the `LineFormatter` provided by Monolog. Open the `config/logging.php` file and locate the channel you are using. Add the following options to the channel configuration:

```php
'formatter' => Monolog\Formatter\LineFormatter::class,
'formatter_with' => [
'format' => "[%datetime%] %channel%.%level_name%: %message%\n",
'dateFormat' => 'Y-m-d g:i:s A',
],
```

This example sets the log format to `[%datetime%] %channel%.%level_name%: %message%\n` and specifies the date format as `Y-m-d g:i:s A`. Feel free to adjust the format according to your requirements.

## Using the Logger

Once you have customized the logger, you can start using it in your Laravel application. The logger is accessible through the `Log` facade.

### Logging Messages

To log messages at different severity levels, you can use the methods provided by the `Log` facade, such as `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`, and `debug`.

Here's an example of logging an info message:

```php
use Illuminate\Support\Facades\Log;

Log::info('User created: John Doe');
```

### Contextual Logging

You can also log additional contextual information along with your messages. The contextual data is typically an array of key-value pairs.

```php
Log::info('User created', ['user_id' => 123]);
```

### Viewing Real-Time Log Updates

To monitor the log file in real-time within your Laravel application, you can utilize the watch and cat Unix commands. These commands are useful during development and debugging.

1.  Open a terminal window and navigate to your Laravel project's root directory.
2.  Use the following command to continuously display the updates in the log file:

```sh
watch -n 1 cat storage/logs/laravel.log
```

    	This command will update the log output every 1 second.

Now, as your Laravel application runs and logs messages, you'll see the real-time updates in the terminal window.

### Conclusion

Customizing and effectively using the logger in your Laravel application is essential for monitoring, debugging, and recording important information. By modifying the log channel and format
