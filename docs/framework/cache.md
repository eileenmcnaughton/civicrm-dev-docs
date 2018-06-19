# Cache Reference

## Using the cache

`Civi::cache()` is the simplest way to access the cache, automatically using the default cache type (described below). The `CRM_Utils_Cache_Interface` class lays out the methods for saving and retrieving cached items.

### Methods

* Set a cache value

    ```php
    Civi::cache()->set('mykey', 'myvalue');
    ```

* Get a cached value

    ```php
    Civi::cache()->get('mykey'); // returns the value, or NULL if not set
    ```

* Delete a cached value

    ```php
    Civi::cache()->delete('mykey');
    ```

* Flush the entire cache

    ```php
    Civi::cache()->flush();
    ```

### Example

```php
/**
 * Finds the magic number, selecting one if necessary.
 *
 * @return int $magicNumber
 *   a magic number between 1 and 100
 */
function findMagicNumber() {
  $magicNumber = Civi::cache()->get('magicNumber');
  if (!$magicNumber) {
    $magicNumber = rand(1,100);
    Civi::cache()->set('magicNumber', $magicNumber);
  }
  return $magicNumber;
}
```

## Cache types

This is selected in `civicrm.settings.php`, where `CIVICRM_DB_CACHE_CLASS` is defined.

* "ArrayCache" - This is the default, using an in-memory array.

* "Memcache" - This is for the PHP Memcache extension.

* "Memcached" - This is for the PHP Memcached extension.

* "APCcache" - This is for the PHP APC extension.

* "NoCache" - This caches nothing

* "Redis" - Redis caching requires that you install and start a Redis Server. [linux install link](https://redis.io/topics/quickstart)
   [mac install link](https://medium.com/@djamaldg/install-use-redis-on-macos-sierra-432ab426640e)
   and that you have a redis php library. [linux install link](https://anton.logvinenko.site/en/blog/how-to-install-redis-and-redis-php-client.html), [mac install link](https://github.com/panxianhai/php-redis-mamp). Don't forget to restart your web service!
   If using Redis in a development environment you may find it easier to declare it conditionally so it copes better if you
   swap between php version or other configs.
  ```
  if (!defined('CIVICRM_DB_CACHE_CLASS')) {
    if (class_exists('Redis')) {
      define('CIVICRM_DB_CACHE_CLASS', 'Redis');
    }
    else {
      define('CIVICRM_DB_CACHE_CLASS', 'ArrayCache');
    }
  }
```
