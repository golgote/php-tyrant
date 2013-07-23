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


# Tyrant_Query options


**Field contains:** *La Geste des Princes-Démons*

## Tyrant_Query::QCSTREQ

This one will work only if the query string is equal to the field. It is also case-sensitive.

## Tyrant_Query::QCSTRINC

Works like *LIKE* in SQL with % before and after the query string, but is also case-sensitive.

- **Princes-D** => La Geste des **Princes-D**émons
- **Princes-d** => *not found*
- **Princes D** => *not found*
- **s Princes-D** => La Geste de**s Princes-D**émons

## Tyrant_Query::QCSTRBW

Is for Begins With. Will find a field that begins with the query string. It is case-sensitive.

Note also that using the QCNEGATE flag, it is possible to find those records which have an empty field. For example :
<pre><code>
$q = new Tyrant_query();
$->addCond('myfield', Tyrant_Query::QCSTRBW | Tyrant_Query::QCNEGATE, '');
</code></pre>

## Tyrant_Query::QCSTREW

Is for Ends With. Will find a field that ends with the query string. It is case-sensitive.

## Tyrant_Query::QCSTRAND

Will find a result if the field includes all tokens in the query string. So Tyrant first tokenizes the query string by removing whitespaces and some other characters, then tries to find records where the queried field contains every tokens from the query string. It is case-sensitive.

- **La des** => **La** Geste **des** Princes-Démons
- **La de** => *not found* because Tyrant uses tokens of words
- **La,Geste** => **La** **Geste** des Princes-Démons
- **La geste** => *not found* because it is case-sensitive
- **La,geste,Geste** => **La** **Geste** des Princes-Démons
- **La Princes** => *not found* because Tyrant doesn't tokenize at -
- **Geste Princes-Démons** => La **Geste** des **Princes-Démons**

## Tyrant_Query::QCSTROR

Like the previous one, Tyrant first tokenizes the query string, then tries to find a record with at least one token from the query string tokens in the queried field. Still is case-sensitive.

- **La des** => **La** Geste **des** Princes-Démons
- **La de** => **La** Geste des Princes-Démons
- **La,Geste** => **La** **Geste** des Princes-Démons
- **La geste** => **La** Geste des Princes-Démons
- **La Princes** => **La** Geste des Princes-Démons
- **Geste Princes-Démons** => La **Geste** des **Princes-Démons**

## Tyrant_Query::QCSTROREQ

This one is a bit tricky because it works like QCSTROR except that it will only find fields that are equals to one of the tokens from our query string. For example, if you have a field *type* which contains either *book*, *author*, *publisher* and you want to find every *books* or *authors*, this is the condition you would probably use.

<pre><code>
$q = new Tyrant_query();
$q->addCond('type', Tyrant_Query::QCSTROREQ, 'book,author');
</code></pre>

## Tyrant_Query::QCSTRRX

Uses a regular expression.

----

## Tyrant_Query::QCFTSOR

Tyrant will first turn the query string into tokens, then it will try to find records which match at least one of these tokens. But unlike QCSTROR, this is done in a case-insensitive way and works on substrings. We could say it works like a case-sensitive LIKE. Note that it only works on ascii. So for example :

- **l** => **L**a Geste des Princes-Démons
- **rin** => La Geste des P**rin**ces-Démons
- **demon** => La Geste des Princes-**Démon**s
- **démon** => *not found* because it only works with ascii
