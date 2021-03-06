# Connections

Connection configuration is the first place you need to start when working with models. li3 supports a number of different relational and non-relational databases, and each is defined through the connection configuration.

Connections are configured in the core bootstrap file `app/config/bootstrap/connections.php`. The `Connections::add()` method is used in this file to make different connections available to your models. 

Each connection is made up of two parts: the first part is the name that identifies the connection. This must be unique, and can be used to fetch connection details and configure models. If you create a connection named "default", your models will use it unless configured otherwise.

The second part of a connection is an array of configuration options specific to the database type you're planning to use. Each connection configuration may differ, but most will include the following:

- `'type'` _string_: The type of data source that defines this connection; typically a
  class or namespace name. Relational database data sources, use `'database'`, while
  CouchDB and other HTTP-related data sources use `'http'`, etc. For classes which
  directly extend `lithium\data\Source`, and do not use an adapter, simply use the
  name of the class, i.e. `'MongoDb'`.
  
- `'adapter'` _string_: For `type`s such as `'database'` which are adapter-driven,
  provides the name of the adapter associated with this configuration.
  
- `'host'` _string_: The host name that the database should connect to. Typically
  defaults to `'localhost'`.
  
- `'login'` _string_: If the connection requires authentication, specifies the login
  name to use.
  
- `'password'` _string_: If the connection requires authentication, specifies the
  password to use.

A quick look at the examples in the default `connections.php` file illustrate the possibilities:

```php
Connections::add('default', array(
	'type' => 'MongoDb',
	'host' => 'localhost',
	'database' => 'project'
));

Connections::add('default', array(
	'type' => 'http',
	'adapter' => 'CouchDb',
	'host' => 'localhost',
	'database' => 'project'
));

Connections::add('default', array(
	'type' => 'database',
	'adapter' => 'MySql',
	'host' => 'localhost',
	'login' => 'root',
	'password' => '',
	'database' => 'project',
	'encoding' => 'UTF-8'
));
```

## Models and Connections

Models will look for the `'default'` connection first. If you've got more than one connection you're using, or wish a model to use an alternate, specify the connection name in the model's `$_meta` property.

```php
class Posts extends \lithium\data\Model {

	protected $_meta = array(
		'connection' => 'legacy'
	);
}
```

For connection-less models you may disable the connection altogether by setting the connection to use to `false`.

```php
class Movies extends \lithium\data\Model {

	protected $_meta = array(
		'connection' => false
	);
}
```

If you ever need to access the connection for model directly, you may do so by using the `connection()` method. The following example shows how to make use of this feature to control PDO transactions.

```php
$source = Posts::connection(); // Gives you the connected data source i.e. a `Database` object.
$pdo = $source->connection; // Gives you the underlying connection of that object.

$pdo->beginTransaction();
$pdo->commit();
$pdo->rollback();
```

## Connections and Environments

Many applications use different databases depending on which environment the application is currently being hosted from. For example, you might want to switch your MongoDB connection from a locally hosted database in development, but use a cloud-based hosting service once you're in production. Connection configurations were built with this in mind.

Once your environments have been defined, use their names as keys in the configuration array, as shown here:

```php
Connections:add('default', array( // 'default' is the name of the connection
	'development' => array(
		'type'     => 'MongoDb',
		'host'     => 'localhost',
		'database' => 'myapp'
	),
	'production' => array(
		'type'     => 'MongoDb',
		'host'     => 'flame.mongohq.com:27111',
		'database' => 'myapp',
		'login'    => 'myuser',
		'password' => 'mysecret'
	)	
));
```
