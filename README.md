# Connection

Currently, PHP Tyrant supports the two API proposed by Tokyo Tyrant, that is [[RDB]] for hash and b-tree databases, and [[RDBTable]] for Table databases.

You connect to your database server using its hostname and port.

<pre><code>
$tt = Tyrant::connect('localhost', 1978);
</code></pre>

In case you want to use multiple connections at the same time, PHP Tyrant keeps an internal pointer to them. It is also possible to connect to the same database server multiple times.

<pre><code>
$tt1 = Tyrant::connect('localhost', 1978, 0);
$tt2 = Tyrant::connect('localhost', 1978, 1);
$tt3 = Tyrant::connect('ttserve.local', 1978);
$tt3 = Tyrant::connect('192.168.0.10', 1980);
</code></pre>

In order to determine the database type your connection uses, PHP Tyrant will first query your server once it is connected, then load the appropriate driver, [[RDB]] or [[RDBTable]].

# Key-value API

Depending on your database type, adding records to the database will be different. Below are some basic operations for dealing with the [[RDB]] type.

## Adding data

There are different ways to add records to the database. Tokyo Tyrant provides the following functions: put, putkeep, putcat, putshl and putnr. Example:

<pre><code>
$tt->put('key', 'value');
</code></pre>

PHP Tyrant also implements the ArrayAccess interface so it is possible to write the above like this:

<pre><code>
$tt['key'] = 'value';
</code></pre>

## Getting data

<pre><code>
$value = $tt['key'];
</code></pre>

## Deleting records

Records can be deleted by key.

<pre><code>
$tt->out('key');
</code></pre>

Tokyo Tyrant also provides a way to delete every records in the database.

<pre><code>
$tt->vanish();
</code></pre>

## Iterator

PHP Tyrant implements the Iterator interface so it is possible to iterate over the records using *foreach*.

<pre><code>
foreach ($tt as $k => $v) {
    var_dump($k, $v);
}
</code></pre>

Tokyo Tyrant also provides a more native interface which might be a little faster, although a little less convenient.

<pre><code>
$tt->iterinit();
while ($k = $tt->iternext()) {
    var_dump($k, $tt->get($k));
}
</code></pre>


# How to use the RDBTable API

RDBTable works like standard database tables except you are not limited by a schema. A key can hold multiple columns, it is up to you to choose which columns are useful for your application.

## Storing records

To store a new record, you decide what is your primary key and what are your data :

<pre><code>
$tt = Tyrant::connect();
$key = 'user:1';
$cols = array('name' => 'john', 'age' => 20);
$tt->put($key, $cols);
</code></pre>

Since the RDBTable API implements ArrayAccess, you can also store a record like this:

<pre><code>
$tt['user:1'] = array('name' => 'john', 'age' => 20);
</code></pre>

The RDBTable API also provides putkeep(). If a record with the same key exists in the database, this method has no effect.

There is also a putcat() to concatenate columns of the existing record. If there is no corresponding record, a new record is created.

## Getting records

To get the record you either use get() or the ArrayAccess API.

<pre><code>
$user = $tt['user:1'];
$user = $tt->get('user:1');
</code></pre>

Searching in the database is performed using a [[TyrantQuery]] object.

