# sqlite-minutils

Added a quick hack to make it work with turso.

Code not working yet.

<!-- WARNING: THIS FILE WAS AUTOGENERATED! DO NOT EDIT! -->

> [!TIP]
>
> ### Where to find the complete documentation for this library
>
> If you want to learn about everything this project can do, we
> recommend reading the Python library section of the sqlite-utils
> project
> [here](https://sqlite-utils.datasette.io/en/stable/python-api.html).
>
> This project wouldn’t exist without Simon Willison and his excellent
> [sqlite-utils](https://github.com/simonw/sqlite-utils) project. Most
> of this project is his code, with some minor changes made to it.


## Use

First, import the sqlite-miniutils library. Through the use of the
**all** attribute in our Python modules by using `import *` we only
bring in the `Database`, `Queryable`, `Table`, `View` classes. There’s
no risk of namespace pollution.

``` python
from sqlite_minutils.db import *
```

Then we create a SQLite database. For the sake of convienance we’re
doing it in-memory with the `:memory:` special string. If you wanted
something more persistent, name it something not surrounded by colons,
`data.db` is a common file name.

``` python
db = Database(":memory:")
```

Let’s drop (aka ‘delete’) any tables that might exist. These docs also
serve as a test harness, and we want to make certain we are starting
with a clean slate. This also serves as a handy sneak preview of some of
the features of this library.

``` python
for t in db.tables: t.drop()
```

User tables are a handy way to create a useful example with some
real-world meaning. To do this, we first instantiate the `users` table
object:

``` python
users = Table(db, 'Users')
users
```

    <Table Users (does not exist yet)>

The table doesn’t exist yet, so let’s add some columns via the
`Table.create` method:

``` python
users.create(columns=dict(id=int, name=str, age=int))
users
```

    <Table Users (id, name, age)>

What if we need to change the table structure?

For example User tables often include things like password field. Let’s
add that now by calling `create` again, but this time with
`transform=True`. We should now see that the `users` table now has the
`pwd:str` field added.

``` python
users.create(columns=dict(id=int, name=str, age=int, pwd=str), transform=True, pk='id')
users
```

    <Table Users (id, name, age, pwd)>

``` python
print(db.schema)
```

    CREATE TABLE "Users" (
       [id] INTEGER PRIMARY KEY,
       [name] TEXT,
       [age] INTEGER,
       [pwd] TEXT
    );

## Queries

Let’s add some users to query:

``` python
users.insert(dict(name='Raven', age=8, pwd='s3cret'))
users.insert(dict(name='Magpie', age=5, pwd='supersecret'))
users.insert(dict(name='Crow', age=12, pwd='verysecret'))
users.insert(dict(name='Pigeon', age=3, pwd='keptsecret'))
users.insert(dict(name='Eagle', age=7, pwd='s3cr3t'))
```

    <Table Users (id, name, age, pwd)>

A simple unfiltered select can be executed using `rows` property on the
table object.

``` python
users.rows
```

    <generator object Queryable.rows_where at 0x10849f6f0>

Let’s iterate over that generator to see the results:

``` python
[o for o in users.rows]
```

    [{'id': 1, 'name': 'Raven', 'age': 8, 'pwd': 's3cret'},
     {'id': 2, 'name': 'Magpie', 'age': 5, 'pwd': 'supersecret'},
     {'id': 3, 'name': 'Crow', 'age': 12, 'pwd': 'verysecret'},
     {'id': 4, 'name': 'Pigeon', 'age': 3, 'pwd': 'keptsecret'},
     {'id': 5, 'name': 'Eagle', 'age': 7, 'pwd': 's3cr3t'}]

Filtering can be done via the `rows_where` function:

``` python
[o for o in users.rows_where('age > 3')]
```

    [{'id': 1, 'name': 'Raven', 'age': 8, 'pwd': 's3cret'},
     {'id': 2, 'name': 'Magpie', 'age': 5, 'pwd': 'supersecret'},
     {'id': 3, 'name': 'Crow', 'age': 12, 'pwd': 'verysecret'},
     {'id': 5, 'name': 'Eagle', 'age': 7, 'pwd': 's3cr3t'}]

We can also `limit` the results:

``` python
[o for o in users.rows_where('age > 3', limit=2)]
```

    [{'id': 1, 'name': 'Raven', 'age': 8, 'pwd': 's3cret'},
     {'id': 2, 'name': 'Magpie', 'age': 5, 'pwd': 'supersecret'}]

The `offset` keyword can be combined with the `limit` keyword.

``` python
[o for o in users.rows_where('age > 3', limit=2, offset=1)]
```

    [{'id': 2, 'name': 'Magpie', 'age': 5, 'pwd': 'supersecret'},
     {'id': 3, 'name': 'Crow', 'age': 12, 'pwd': 'verysecret'}]

The `offset` must be used with `limit` or raise a `ValueError`:

``` python
try:
    [o for o in users.rows_where(offset=1)]
except ValueError as e:
    print(e)
```

    Cannot use offset without limit
