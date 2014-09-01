# Doctrine ORM Service Provider for Silex
This extension sets up Doctrine ORM for Silex by reusing the database connection established with the DBAL extension (the default DoctrineServiceProvider included with Silex).

You'll find the following topics in this README:
*   [Dependencies](#dependencies)
*   [Installation](#installation)
*   [Configuration](#configuration)
*   [Usage](#usage)

## Dependencies
*   [Silex](http://silex.sensiolabs.org/)
    _(Not mentioned in composer.json, because -in my opinion- it's a direct dependency of your own project, so you should put it in your own JSON-file.)_
*   PHP 5.3.2 and above
*   [Doctrine 2 Object Relational Mapper](https://github.com/doctrine/doctrine2)
*   [Doctrine Database Abstraction Layer](https://github.com/doctrine/dbal)
*   [Common library for Doctrine](https://github.com/doctrine/common)


## Installation
You basically have three options to add this extension to your Silex project. We'd strongly recommend the first option!

### composer.json
http://packagist.org/packages/fredjuvaux/silex-orm-provider

Add 'fredjuvaux/silex-orm-provider' to the dependencies in your projects composer.json file and update your dependencies.
This is by far the easiest way, since it automatically adds the Doctrine dependencies and adds everything to the autoloading mechanism supplied by Composer.

More information on Composer can be found on [getcomposer.org](http://getcomposer.org/).

### Git
Another option is to clone the project:

```bash
cd /path/to/your_project/vendor
git clone git@github.com:fredjuvaux/silex-orm-provider.git
```

Or you can add it as a submodule if your project is in a git repository too:

```bash
cd /path/to/your_project
git submodule add git@github.com:fredjuvaux/silex-orm-provider.git vendor/silex-orm-extension
```

This will require you to manually install all the dependencies. Also note that you'll need to add the provider to the Silex autoloader (or whatever autoloading mechanism) by hand. More on both subjects can be found below.

### Download an archive
GitHub also gives you the option to [download an ZIP-archive](https://github.com/fredjuvaux/silex-orm-provider/zipball/master), which you can extract in your vendor folder. This method will also require you to manually install all the dependencies and add everything to your autoloader.


## Configuration
First of all you should have the Doctrine DBAL connection configured. For more information about configuring the DoctrineServiceProvider, I'd recommend reading [this page of the Silex documentation](http://silex.sensiolabs.org/doc/providers/doctrine.html).

Registering the Doctrine ORM Service Provider is rather straight forward:

```php
<?php

/* ... */

$app['autoloader']->registerNamespace('SilexORM', __DIR__ . '/vendor/silex-orm-extension/lib');

$app->register(new SilexORM\Provider\DoctrineORMServiceProvider(), array(
    'db.orm.class_path'            => __DIR__.'/vendor/doctrine/lib',
    'db.orm.proxies_dir'           => __DIR__.'/var/cache/doctrine/Proxy',
    'db.orm.proxies_namespace'     => 'DoctrineProxy',
    'db.orm.auto_generate_proxies' => true,
    'db.orm.entities'              => array(array(
        'type'      => 'annotation',
        'path'      => __DIR__.'/app',
        'namespace' => 'Entity',
    )),
));

/* ... */

```

**Note:** If you don't want to use the internal autoloader of Silex (because you're using another one, like the one generated by composer for example), you can leave out the line starting with ``$app['autoloader']`` and the _db.orm.class_path_ parameter.

### Parameters (and their default values)
Below you'll find all the parameters with their defaults (they can also be found in [DoctrineORMServiceProvider::setOrmDefaults](https://github.com/fredjuvaux/silex-orm-provider/blob/master/lib/SilexORM/Provider/DoctrineORMServiceProvider.php#L48-68)).

*   __db.orm.auto_generated_proxies__
    
    _Default:_ ``true``
    
    Sets whether proxy classes should be generated automatically at runtime by Doctrine.
    If set to ``false``, proxy classes must be generated manually using the Doctrine command line tools.
    It is recommended to disable autogeneration for a production environment.

*   __db.orm.cache__

    _Default:_ ``new ArrayCache``

    Defines what caching method should be used. The default (ArrayCache) should be fine when you're developing, but in a production environment you probably want to change this to something like APC or Memcache.
    More information can be found in [Chapter 22. Caching](http://docs.doctrine-project.org/projects/doctrine-orm/en/2.1/reference/caching.html "Doctrine 2 ORM 2.1 Documentation") of the Doctrine 2 ORM 2.1 documentation.

*   __db.orm.class_path__

    You only need to use this if you want to autoload the Doctrine 2 ORM classes using the Silex autoloader.
    It should point to the folder where the classes is stored (the lib folder on the Doctrine repository).

*   __db.orm.entities__

    _Default:_
```php
array(
    array(
        'type' => 'annotation',
        'path' => 'Entity',
        'namespace' => 'Entity',
    )
)
```

    An array of arrays which should contain:
    *   ``type``: Type of metadata driver used (``annotation``, ``yml``, ``xml``)
    *   ``path``: Path to where the metadata is stored
    *   ``namespace``: Namespace used for the entities

    The Doctrine ORM Service Provider uses a _DriverChain_ internally to configure Doctrine 2 ORM.
    This allows you to use mixed types of metadata drivers in a single project.

*   __db.orm.proxies_dir__

    _Default:_ ``cache/doctrine/Proxy``

    Sets the directory where Doctrine generates any proxy classes.
    For a detailed explanation on proxy classes and how they are used in Doctrine, refer to [section 3.5 Proxy Objects](http://docs.doctrine-project.org/projects/doctrine-orm/en/2.1/reference/configuration.html#proxy-objects) of the Doctrine ORM documentation.

*   __db.orm.proxies_namespace__

    _Default:_ ``DoctrineProxy``

    Sets the namespace to use for generated proxy classes.
    For a detailed explanation on proxy classes and how they are used in Doctrine, refer to [section 3.5 Proxy Objects](http://docs.doctrine-project.org/projects/doctrine-orm/en/2.1/reference/configuration.html#proxy-objects) of the Doctrine ORM documentation.


## Usage
You can access the EntityManager by calling ``$app['db.orm.em']``.
