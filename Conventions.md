# 默认值

因为我们在配置项已经接受了一个默认值，所以使用我们的库并不难。这些默认值很容易记住并且也有助于开发者保证开发的流畅性。

如果你已经看过了配置/设置部分，那么你应该知道这并不多。因此，使用php-activerecord主要需要你让自己熟悉一些简单的默认值。一旦你做到了这一点，你可以学习一些更高级的特性。

ActiveRecord假定以下的默认值：

```php
# name of your class represents the singular form of your table name.
class Book extends ActiveRecord\Model {}

# your table name would be "people" 
class Person extends ActiveRecord\Model {}
```

数据库表的主键名默认为”id”。

```php
class Book extends ActiveRecord\Model {}

# SELECT * FROM `books` where id = 1
Book::find(1);
```

## 修改默认值

尽管ActiveRecord偏向于使用默认的表名和主键名，但是你能够覆盖他们。下面是一个如何配置特殊的模型的简单的例子。

```php
class Book extends ActiveRecord\Model
{
  # explicit table name since our table is not "books" 
  static $table_name = 'my_book';

  # explicit pk since our pk is not "id" 
  static $primary_key = 'book_id';

  # explicit connection name since we always want our test db with this model
  static $connection = 'test';

  # explicit database name will generate sql like so => my_db.my_book
  static $db = 'my_db';
}
```

