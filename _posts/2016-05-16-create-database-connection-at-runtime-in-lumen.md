---
layout: post
title: Lumen - Create database connection at runtime
categories: [lumen]
comments: true
tags: [lumen, framework, database-connection, runtime]
feature: assets/media/2016-05-16/lumen-database-connection-post-image.png
excerpt: How to create a database connection at runtime and save it in config object
---

[Lumen](https://https://lumen.laravel.com/) is a great micro-framework for building staless RESTful APIs. It inherits concepts from its bigger brother [Laravel](https://laravel.com) but tries to stay slim.
Considering the background of Laravel you might notice some things do not work as you would imagine - when coming from Laravel development.

### Problem

Recently I came across a [good question](http://stackoverflow.com/questions/37215265/lumen-create-database-connection-at-runtime/37243775#37243775) on StackOverflow how to create a database connection at runtime, i. e. for a certain route.

As you might think (_That's easy_) you would probably go to your `app/Http/routes.php` and add the following:

{% highlight php %}
$app->get('/', function () use ($app) {
    $config = $app->make('config');
    $config->set('database.connections.retail_db', [
        'driver'   => 'mysql',
        'host'     => env('RETAIL_DB_HOST', 'localhost'),
        'port'     => env('RETAIL_DB_PORT', 5432),
        'database' => env('RETAIL_DB_DATABASE', 'forge'),
        'username' => env('RETAIL_DB_USERNAME', 'forge'),
        'password' => env('RETAIL_DB_PASSWORD', ''),
        'charset'  => env('RETAIL_DB_CHARSET', 'utf8'),
        'prefix'   => env('RETAIL_DB_PREFIX', ''),
        'schema'   => env('RETAIL_DB_SCHEMA', 'public'),
    ]);
    return app('db')->connection('retail_db')->select("SELECT * FROM users");
});
{% endhighlight %}

When running this it's very likely you the following error:

> InvalidArgumentException in DatabaseManager.php line 239:  
> Database [retail_db] not configured.

Obviously there is not configuration object loaded on which we can set the new database connection - echoing `$config->all()` will tell us the same.

### Solution

To initialise the database configuration (and load some preconfigured) connections you need to do the following:

- Go to your `app/bootstrap/app.php` and add `$app->configure('database');`, i. e. on line 29
- _Optional:_
  - Create a new folder `app/config`
  - Copy a sample `database.php` from `vendor/laravel/lumen-framework/config/database.php` to the new `config` folder

After that your new database connection `retail_db` will work as you'd expect it. To remove your newly created connection just use `$config->set('database.connections.retail_db', null);`

If you got question or do have any remarks just comment or drop me a message!





