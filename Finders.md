# 查找方式

ActiveRecord支持许多查找记录的方法，例如：通过主键，动态字段名词查找等方式。它有能力通过一个简单的调用来获取一张表中的所有记录，你也能够使用一些选项例如order,limit,select和group。

本质上来讲，这里有两种可用的查找类型：一条单记录结果和多记录结果集有时候这些方法调用会有些透明，这意味着你可以使用相同的方法来获取另一个，但是你需要传递一个选项给这个方法来区分你获取的结果类型。

所有的方法常常使用Model::find()方法来从你的数据库里面获取数据，只有一个例外，即通常的sql查询使用Model::find_by_sql()方法。在所有的情况中，在ActiveRecord中的查询方法都是被静态调用的。这意味着你总是使用以下的语法类型。

```php
class Book extends ActiveRecord\Model {}

Book::find('all');
Book::find('last');
Book::first();
Book::last();
Book::all();
```

## 单记录结果

不论何时你调用产生单记录结果的方法，这个方法都会返回一个你模型类的实例。这里有三种不同的方法来获取单记录结果。我们从最基础的形式开始。

### 主键查找

你能够通过传递一个主键给查询方法来获取一条记录。你也可以传递一个选项数组作为第二个参数来创建一个特殊的查询。如果没有找到任何记录，会抛出一个RecordNotFound错误。

```php
# Grab the book with the primary key of 2
Book::find(2);
# sql => SELECT * FROM `books` WHERE id = 2
```

### 查找首条记录

你能够通过两种方法从你的数据库中获取首条记录。如果你没有传递第二个参数作为条件，这个方法会取你数据库中的所有记录，但是至少返回第一个结果给你。如果没有找到任何记录则会返回Null。

```php
# Grab all books, but only return the first result back as your model object.
$book = Book::first();
echo "the first id is: {$book->id}" # => the first id is: 1
# sql => SELECT * FROM `books`

# this produces the same sql/result as above
Book::find('first');
```

### 查找末尾记录

如果你在看这个指南的时候没有睡着，那么你应该猜到了这个和查找首条记录相同，除了它会返回最后一个结果之外。如果没有任何记录将会返回Null。

```php
# Grab all books, but only return the last result back as your model object.
$book = Book::last();
echo "the last id is: {$book->id}" # => the last id is: 32
# sql => SELECT * FROM `books`

# this produces the same sql/result as above
Book::find('last');
```

## 多记录结果集

这个类型的结果总是返回一个模型对象的数组。如果你的表中没有记录，或者你的查询结果中没有结果，那么将会返回一个空数组。

### 通过主键查询

和单记录结果的主键查询相同，你能够传递一个包含多个主键的数组给查询函数。或者，你能够传递一个选项数组作为最后一个参数来创建特殊的查询。每一个你使用的键作为一个参数移动会产生一条相对应的记录，否则会抛出RecordNotFound错误。

```php
# Grab books with the primary key of 2 or 3
Book::find(2,3);
# sql => SELECT * FROM `books` WHERE id IN (2,3)

# same as above
Book::find(array(2,3));
```

### 获取所有记录

这里有两种甚至更多的方法能从你的数据库中获取多条记录。他们使用不同的方式。然而，他们基本上是相同的。如果你没有传递选项数组，那么它会获取所有记录。

```php
# Grab all books from the database
Book::all();
# sql => SELECT * FROM `books`

# same as above
Book::find('all');
```

现在我们传递一些选项值给同样的方法，这样我们不会获取每一条记录。

```php
$options = array('limit' => 2);
Book::all($options);
# sql => SELECT * FROM `books` LIMIT 0,2

# same as above
Book::find('all', $options);
```

## 查询选项

这里有很多可用的选项能够传递给查询方法中的某一个。让我们先从其中最重要的选项之一：Conditions开始。

## Conditions

这就是”WHERE”在SQL语句中的条件。通过添加条件，ActiveRecord会把他们理解成SQL语句中的”WHERE”从而来筛选你的结果。条件能够支持使用非常简单的字符串。他们也能够变得复杂，当你想创建条件语句那样使用?标志作为占位符来填充数值。让我们从一个简单的条件字符串的简单例子开始。

```php
# fetch all the cheap books!
Book::all(array('conditions' => 'price < 15.00'));
# sql => SELECT * FROM `books` WHERE price < 15.00

# fetch all books that have "war" somewhere in the title
Book::find('all', array('conditions' => "title LIKE '%war%'"));
# sql => SELECT * FROM `books` WHERE title LIKE '%war%'
```

作为规定，你能够使用*?*标志作为数值的位符，ActiveRecord将会使用你所提供的数值来替换它们。使用这个步骤的方法的好处是ActiveRecord会在后端使用你的数据库本地函数编码你的字符串，从而避免SQL注入。

```php
# fetch all the cheap books!
Book::all(array('conditions' => array('price < ?', 15.00)));
# sql => SELECT * FROM `books` WHERE price < 15.00

# fetch all lousy romance novels
Book::find('all', array('conditions' => array('genre = ?', 'Romance')));
# sql => SELECT * FROM `books` WHERE genre = 'Romance'

# fetch all books with these authors
Book::find('all', array('conditions' => array('author_id in (?)', array(1,2,3))));
# sql => SELECT * FROM `books` WHERE author_id in (1,2,3)

# fetch all lousy romance novels which are cheap
Book::all(array('conditions' => array('genre = ? AND price < ?', 'Romance', 15.00)));
# sql => SELECT * FROM `books` WHERE genre = 'Romance' AND price < 15.00
```

下面是一些更复杂的例子。再次说明，第一个条件数组的索引是条件字符串。数组中的值在那之后将会用来替换？标记。

```php
# fetch all cheap books by these authors
$cond =array('conditions'=>array('author_id in(?) AND price < ?', array(1,2,3), 15.00));
Book::all($cond);
# sql => SELECT * FROM `books` WHERE author_id in(1,2,3) AND price < 15.00
```

## Limit&Offset

这个应该相当明显。一个限制选项会产生一个用于数据库的SQL限制子句。它能够和补偿选项结合使用。

```php
# fetch all but limit to 10 total books
Book::find('all', array('limit' => 10));
# sql => SELECT * FROM `books` LIMIT 0,10

# fetch all but limit to 10 total books starting at the 6th book
Book::find('all', array('limit' => 10, 'offset' => 5));
# sql => SELECT * FROM `books` LIMIT 5,10
Order
产生一个ORDER BY SQL子句。
# order all books by title desc
Book::find('all', array('order' => 'title desc'));
# sql => SELECT * FROM `books` ORDER BY title desc

# order by most expensive and title
Book::find('all', array('order' => 'price desc, title asc'));
# sql => SELECT * FROM `books` ORDER BY price desc, title asc
```

## Order

产生一个有序的SQL查询。

```php
# order all books by title desc
Book::find('all', array('order' => 'title desc'));
# sql => SELECT * FROM `books` ORDER BY title desc

# order by most expensive and title
Book::find('all', array('order' => 'price desc, title asc'));
# sql => SELECT * FROM `books` ORDER BY price desc, title asc
```

## Select

在你的选项数组中传递一个选择的键会允许你指定你需要从数据库中返回的列。当你的表中有太多列，但是你只能想50条记录中获取3列的时候。这是非常有用的。当你使用group语句的时候，它也是非常有用的。

```php
# fetch all books, but only the id and title columns
Book::find('all', array('select' => 'id, title'));
# sql => SELECT id, title FROM `books`

# custom sql to feed some report
Book::find('all', array('select' => 'avg(price) as avg_price, avg(tax) as avg_tax'));
# sql => SELECT avg(price) as avg_price, avg(tax) as avg_tax FROM `books` LIMIT 5,10
```

## From

这用来指定你从那张表进行选择。如果你需要一个更好的控制这个能变得便捷。

```php
# fetch the first book by aliasing the table name
Book::first(array('select'=> 'b.*', 'from' => 'books as b'));
# sql => SELECT b.* FROM books as b LIMIT 0,1
```

## Group

生成一个GROUP BY子句。

```php
# group all books by prices
Book::all(array('group' => 'price'));
# sql => SELECT * FROM `books` GROUP BY price
```

## Having

生成一个Having子句来向你的GROUP BY中添加条件。

```php
# group all books by prices greater than $45
Book::all(array('group' => 'price', 'having' => 'price > 45.00'));
# sql => SELECT * FROM `books` GROUP BY price HAVING price > 45.00
```

## Read only

Readonly模型就是：只读。如果你尝试保存一个只读模型，那么会抛出ReadOnlyException错误。

```php
# specify the object is readonly and cannot be saved
$book = Book::first(array('readonly' => true));

try {
  $book->save();
} catch (ActiveRecord\ReadOnlyException $e) {
  echo $e->getMessage();
  # => Book::save() cannot be invoked because this model is set to read only
}
```

## Dynamic finders

这里提供一个快速简便的方法来构建条件选项而不是通过传递一个臃肿的数组选项。这个选项需要使用PHP5.3中的延迟静态绑定，它结合__callStatic()方法能够调用在你的模型中没有定义的静态方法。你也能够使用返回单记录结果的YourModel::find_by方法以及返回多记录结果集的YourModel::find_all方法。你需要做的只是添加一个下划线和另一个列名在这两个方法之后，让我们来看一看。

```php
# find a single book by the title of War and Peace
$book = Book::find_by_title('War and Peace');
#sql => SELECT * FROM `books` WHERE title = 'War and Peace'

# find all discounted books
$book = Book::find_all_by_discounted(1);
#sql => SELECT * FROM `books` WHERE discounted = 1

# find all discounted books by given author
$book = Book::find_all_by_discounted_and_author_id(1, 5);
#sql => SELECT * FROM `books` WHERE discounted = 1 AND author_id = 5

# find all discounted books or those which cost 5 bux
$book = Book::find_by_discounted_or_price(1, 5.00);
#sql => SELECT * FROM `books` WHERE discounted = 1 OR price = 5.00
```

## Joins

一个结合选项将会传递一个指定的SQL JOINS。这里有两个方法来构造一个JOIN语句。你可以传递一个默认的简单的字符串作为SQL语句来执行结合。通常来说，结合选项不会从结合的表中选择属性，而它只会在你的模型表中选择属性。你可以传递一个选择选项来指定列。

```php
# fetch all books joining their corresponding authors
$join = 'LEFT JOIN authors a ON(books.author_id = a.author_id)';
$book = Book::all(array('joins' => $join));
# sql => SELECT `books`.* FROM `books`
#LEFT JOIN authors a ON(books.author_id = a.author_id)
```

或者，你可以通过关联模型来指定一个结合。

```php
class Book extends ActiveRecord\Model
{
   static $belongs_to = array(array('author'),array('publisher'));
}
 
# fetch all books joining their corresponding author
$book = Book::all(array('joins' => array('author')));
# sql => SELECT `books`.* FROM `books`
#INNER JOIN `authors` ON(`books`.author_id = `authors`.id)

# here's a compound join
$book = Book::all(array('joins' => array('author', 'publisher')));
# sql => SELECT `books`.* FROM `books`
#INNER JOIN `authors` ON(`books`.author_id = `authors`.id)
#INNER JOIN `publishers` ON(`books`.publisher_id = `publishers`.id)
```

结合可以与字符串或者关联模型合并。

```php
class Book extends ActiveRecord\Model
{
   static $belongs_to = array(array('publisher'));
}

$join = 'LEFT JOIN authors a ON(books.author_id = a.author_id)';
# here we use our $join string and the association publisher
$book = Book::all(array('joins' => $join, 'publisher'));
# sql => SELECT `books`.* FROM `books`
#LEFT JOIN authors a ON(books.author_id = a.author_id)
#INNER JOIN `publishers` ON(`books`.publisher_id = `publishers`.id)
```

#  通过常规SQL语句查找

如果由于某些原因，你需要创建一个超出了选择选项能力的复杂的SQL查询语句，你可以传递一个常规的SQL查询语句给Model::find_by_sql()方法。这会使你的模型变成只读，这样你不能够使用任何写方法来操作你的模型。

警告：find_by_sql()方法不像其他方法一样预防SQL注入攻击。确保find_by_sql()方法的安全责任全部在于你。你可以使用Model::connection()->escape()来避免SQL字符串。

```php
# this will return a single result of a book model with only the title as an attirubte
$book = Book::find_by_sql('select title from `books`');

# you can even select from another table
$cached_book = Book::find_by_sql('select * from books_cache');
# this will give you the attributes from the books_cache table
```

## 立即加载关联

立即加载恢复基础模型，它之关联很少一部分查询。这就避免了N+1问题。

想象你查找10个公告并且展示每一个公告作者的姓的代码。

```php
$posts = Post::find('all', array('limit' => 10));
foreach ($posts as $post)
  echo $post->author->first_name;
```

这里我们获取了11个查询结果，第1到第10个公告，+10（每一个公告从作者表中获取姓）

我们通过使用只发出两个请求而不是11个的include选项解决这个问题。下面就是我们如何完成的。

```php
$posts = Post::find('all', array('limit' => 10, 'include' => array('author')));
foreach ($posts as $post)
  echo $post->author->first_name;

SELECT * FROM `posts` LIMIT 10
SELECT * FROM `authors` WHERE `post_id` IN (1,2,3,4,5,6,7,8,9,10)
```

因为include使用一个数组，所以你可以指定不止一个的关联。

```php
$posts = Post::find('all', array('limit' => 10, 'include' => array('author', 'comments')));
```

你也可以限制include选项来立即加载关联的关联。

```php
$posts = Post::find('first', array('include' => array('category', 'comments' => array('author'))));
```

