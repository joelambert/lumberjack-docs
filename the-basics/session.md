# Sessions
The HTTP protocol is inherently stateless. If you want to maintain some data between requests, you will need to use the Session.

## Configuration
The Session is configured using the `config/session.php` file. By default Lumberjack is configured to store all session data as files on the disk.

TODO: Outline configuration options and talk about how encryption can be enabled

## Usage

### Retrieving Data
The primary way to work with session data is via the `Session` Facade:

```php
use Rareloop\Lumberjack\Facades\Session;

$value = Session::get('key');
```

You can also pass an optional second parameter to use as a default in the instance the requested key doesn't exist:

```php
use Rareloop\Lumberjack\Facades\Session;

$value = Session::get('key', 'default');
```

#### Retrieving All Session Data
You can use the `all()` function if you wish to retrieve all the session data:

```php
use Rareloop\Lumberjack\Facades\Session;

$data = Session::all();
```

#### Determine If An Item Exists In The Session
You can check if a specific key is stored in the session using the `has()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

if (Session::has('key')) {
    // Do something
}
```

#### Retrieve and remove
If you wish to remove an item from the session but also retrieve it's current value, you can use the `pull()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

$value = Session::pull('key');
```

### Storing Data
To store data into the session you use the `put()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

Session::put('key', 'default');
```

#### Adding Data to Array Session Values
If you have an array in your session, you can add new data to it by using the `push()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

Session::push('key', 'default');
```

### Flash Data
There are times when you only want to keep session data for the next request, e.g. form values when a validation error occurs.

You can achieve this easily in Lumberjack using the `flash()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

Session::flash('key', 'default');
```

If you later decide that you need the data to persist in the session you can do this in two ways. Use the `keep()` to select specific keys to retain or `reflash()` to store all flash data for an additional request.

```php
use Rareloop\Lumberjack\Facades\Session;

Session::keep('key');

// or

Session::reflash();
```

### Deleting Data
To remove data from the session you use the `forget()` function:

```php
use Rareloop\Lumberjack\Facades\Session;

Session::forget('key');
```

## Adding Custom Storage Drivers
Lumberjack is capable of using multiple different drivers for session storage. The default and only driver provided in the core is the `file` driver, which saves all session data to disk.

It is simple to implement and register a new driver though.

### Implement the driver
Start by creating a class that implements the standard PHP [SessionHandlerInterface](http://php.net/manual/en/class.sessionhandlerinterface.php) that provides the storage functionality you require:

```php
namespace App\Session\Drivers;

class DatabaseSessionDriver implements \SessionHandlerInterface
{
    /**
     * Perform and opening/initialisation required for this driver
     * Note: In most cases this can be empty
     */
    public function open ($save_path, $session_name) {}

    /**
     * Perform and closing/shutdown required for this driver
     * Note: In most cases this can be empty
     */
    public function close () {}

    /**
     * Return a string of the data associated with $session_id
     */
    public function read ($session_id) {}

    /**
     * Save the provided string data against $session_id
     */
    public function write ($session_id, $session_data) {}

    /**
     * Remove any data associated with $session_id
     */
    public function destroy ($session_id) {}

    /**
     * Remove all session data that is older than $maxlifetime
     * Note: $maxlifetime is provided as a Unix timestamp
     */
    public function gc ($maxlifetime) {}
}
```

### Register the driver
To register the new driver you need to use a Service Provider. If you don't want to create a new provider you can use the `AppServiceProvider` which can be found in `app\Providers\AppServiceProvider.php`.

Registration is done by calling `extend()` on the Session facade:

```php
<?php

namespace App\Providers;

use Rareloop\Lumberjack\Facades\Session;
use Rareloop\Lumberjack\Providers\ServiceProvider;
use App\Session\Drivers\DatabaseSessionDriver;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any app specific items into the container
     */
    public function register()
    {

    }

    /**
     * Perform any additional boot required for this application
     */
    public function boot()
    {
        Session::extend('database', function($app) {
            return new DatabaseSessionDriver;
        });
    }
}

```

After this it's just a matter of updating the `driver`  value in `config/session.php` to match the key you passed to `extend()`, in this instance this would be `database`.