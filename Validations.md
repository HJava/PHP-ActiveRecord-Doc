# 验证器

验证器提供一个简单而有力的模式来确保你数据的完整性。通过在你的模型中声明验证器，你可以保证只有有效的数据会被保存到你的数据库中。你将不再需要撤销合法性验证邮箱和停止被保存的数据的函数。通过验证器，如果你的数据是无效的，ActiveRecord将会注意标记这个数据为无效的，而不是写进数据库中。

验证器将会与以下的方法正常的结合运行：

```php
$book->save();
Book::create();
$book->update_attributes(array('title' => 'new title'));
```

以下将会跳过验证器而直接保存记录：

```php
$book->update_attribute();
$book->save(false); # anytime you pass false to save it will skip validations
```

## 我的模型有效么？

你可以通过使用以下的任意一个方法来确定你的模型是不是有效的，能够将数据保存到数据库中：Model:is_valid或Model:is_invalid。当被调用时，这两个方法都会运行你模型中的验证器。

```php
class Book extends ActiveRecord\Model
{
  static $validates_presence_of = array(
    array('title')
  );
}

# our book won't pass validates_presence_of
$book = new Book(array('title' => ''));
echo $book->is_valid(); # false
echo $book->is_invalid(); # true
```

如果在你模型中的验证器失效，你将会收到类似这样的错误信息。让我们假设我们的验证器出现[validates_presence_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_presence_of)错误。

```php
class Book extends ActiveRecord\Model
{
  static $validates_presence_of = array(
    array('title')
  );
}
$book = new Book(array('title' => ''));
$book->save();
$book->errors->is_invalid('title'); # => true

# if the attribute fails more than 1 validation,
# you would get an array of errors below

echo $book->errors->on('title'); # => can't be blank
```

现在让我们假设我们的模型的两个验证器失效：[validates_presence_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_presence_of)和[validates_size_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_size_of)。

```php
class Book extends ActiveRecord\Model
{
  static $validates_presence_of = array(
    array('title')
  );

  static $validates_size_of = array(
    array('title', 'within' => array(1,20))
  );
}

$book = new Book(array('title' => ''));
$book->save();
$book->errors->is_invalid('title'); # true

print_r($book->errors->on('title'));

# which would give us:

# Array
# (
#   [0] => can't be blank
#   [1] => is too short (minimum is 1 characters)
# )

# to access errors for all attributes, not just 'title'
print_r($book->errors->full_messages());
```

## 共性

验证器是通过一组常规的选项和一些特殊选项来定义的。只要你看来上面的文档，创建一个验证器是通过在你的模型类声明一个静态的验证变量例如多维数组（来验证多个属性）。每一个验证器需要你用属性名作为数组的索引。你也可以通过作为值的信息字符串来创建一个信息密钥从而配置错误信息。你也可以在创建或者添加的时候添加一个只运行验证器的选项。默认的，你的验证器将会在每次Model#save()调用的时候被触发。

```php
class Book extends ActiveRecord\Model
{
  # 0 index is title, the attribute to test against
  # message is our custom error msg
  # only run this validation on creation - not when updating
  static $validates_presence_of = array(
    array('title', 'message' => 'cannot be blank on a book!', 'on' => 'create')
  );
}
```

在一些验证器中你可以使用：in即within。你通过In/within指定数组中的第一个和第二个元素来代表开始和结束的范围。

所有验证器的验证器常规有效选项：

- on:在保存或者更新的时候运行
- allow_null：验证器允许保留null值。
- allow_black：验证器允许保留空字符串。
- message：指定一个通常的错误信息。

## 有效的验证器

这里有许多你可以在你的模型中指定特殊属性的提前定义的验证例程。

- [validates_presence_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_presence_of)
- [validates_size_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_size_of)
- [validates_length_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_size_of)
- [validates_(in|ex)clusion_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_in_ex_clusion_of)
- [validates_format_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_format_of)
- [validates_numericality_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_numericality_of)
- [validates_uniqueness_of](http://www.phpactiverecord.org/projects/main/wiki/Validations#validates_uniqueness_of)
- [validate*custom](http://www.phpactiverecord.org/projects/main/wiki/Validations#validate)

### validates_presence_of

这大概是最简单的验证器。他确保这个属性值不会是null或者空字符串。有效的选项：

message：默认不能为空字符串。

```php
class Book extends ActiveRecord\Model
{
  static $validates_presence_of = array(
    array('title'),
    array('cover_blurb', 'message' => 'must be present and witty')
  );
}

$book = new Book(array('title' => ''));
$book->save();

echo $book->errors->on('cover_blurb'); # => must be present and witty
echo $book->errors->on('title'); # => can't be blank
```

### validates_size_of / validates_length_of

这两个验证器是同一个。目的是验证给定属性的字符长度。有效的选项：

- is：属性应该正好为n个字符长度。
- in/within：属性值应该在数组(n,m)内。
- maximum/minimum:属性值分别不应该高于/低于。

每一个选项有特殊的信息并且可以被改变。

- is：使用关键字“wrong_length”。
- in/within：使用关键字“too_long”&”too_short”。
- maximum/minimum:使用关键字“too_long”&“too_short”。

```php
class Book extends ActiveRecord\Model
{
  static $validates_size_of = array(
    array('title', 'within' => array(1,5), 'too_short' => 'too short!'),
    array('cover_blurb', 'is' => 20),
    array('description', 'maximum' => 10, 'too_long' => 'should be short and sweet')
  );
}

$book = new Book;
$book->title = 'War and Peace';
$book->cover_blurb = 'not 20 chars';
$book->description = 'this description is longer than 10 chars';
$ret = $book->save();

# validations failed so we get a false return
if ($ret == false)
{
  # too short!
  echo $book->errors->on('title');

  # is the wrong length (should be 20 chars)
  echo $book->errors->on('cover_blurb');

  # should be short and sweet
  echo $book->errors->on('description');
}
```

### validates_(in|ex)clusion_of

如你所看到的名字一样，这两个是相似的。实际上，这只是你验证器的黑/白名单方法。包含是指一个需要在一个给定的集合中的值的白名单。排除是相反的，一个需要给定值不在集合中的黑名单。有效选项：

- in/within：属性应该/不应该是数组中的值。
- message：常规错误信息。

```php
class Car extends ActiveRecord\Model
{
  static $validates_inclusion_of = array(
    array('fuel_type', 'in' => array('petroleum', 'hydrogen', 'electric')),
  );
}

# this will pass since it's in the above list
$car = new Car(array('fuel_type' => 'electric'));
$ret = $car->save();
echo $ret # => true

class User extends ActiveRecord\Model
{
  static $validates_exclusion_of = array(
    array('password', 'in' => array('god', 'sex', 'password', 'love', 'secret'),
      'message' => 'should not be one of the four most used passwords')
  );
}

$user = new User;
$user->password = 'god';
$user->save();

# => should not be one of the four most used passwords
echo $user->errors->on('password');
```

### validates_format_of

这个验证器使用preg_match方法来验证属性的格式。你可以创建一个正则表达式来再次测试。有效属性：

- with：正则表达式


- message：常规错误信息

```php
class User extends ActiveRecord\Model
{
  static $validates_format_of = array(
    array('email', 'with' =>
      '/^[^0-9][A-z0-9_]+([.][A-z0-9_]+)*[@][A-z0-9_]+([.][A-z0-9_]+)*[.][A-z]{2,4}$/')
    array('password', 'with' =>
      '/^.*(?=.{8,})(?=.*\d)(?=.*[a-z])(?=.*[A-Z]).*$/', 'message' => 'is too weak')
  );
}

$user = new User;
$user->email = 'not_a_real_email.com';
$user->password = 'notstrong';
$user->save();

echo $user->errors->on('email'); # => is invalid
echo $user->errors->on('password'); # => is too weak
```

### validates_numericality_of

如名字所示，这个方法检查属性是不是一个数值，是不是一个确定的值、有效的选项：

- only_integer：数值必须是一个数字(注：不是一个单精度浮点数)，信息：“不是一个数字”
- even，message：“必须是偶数”
- odd，message：“必须是奇数”
- greater_than：>，信息：“必须比%d大”
- equal_to：==，信息：“必须与%d相等”
- less_than：<：“信息：必须小于%d”
- less_than_or_equal_to：<=，信息：“必须小于或等于%d”

```php
class Order extends ActiveRecord\Model
{
  static $validates_numericality_of = array(
    array('price', 'greater_than' => 0.01),
    array('quantity', 'only_integer' => true),
    array('shipping', 'greater_than_or_equal_to' => 0),
    array('discount', 'less_than_or_equal_to' => 5, 'greater_than_or_equal_to' => 0)
  );
}

$order = new Order;
$order->price = 0;
$order->quantity = 1.25;
$order->shipping = 5;
$order->discount = 2;
$order->save();

echo $order->errors->on('price'); # => must be greater than 0.01
echo $order->errors->on('quantity'); # => is not a number
echo $order->errors->on('shipping'); # => null
echo $order->errors->on('discount'); # => null
```

### validates_uniqueness_of

测试给定的属性是不是已经存在于表格中。

- message：常规错误信息

```php
class User extends ActiveRecord\Model
{
  static $validates_uniqueness_of = array(
    array('name'),
    array(array('blah','bleh'), 'message' => 'blah and bleh!')
  );
}

User::create(array('name' => 'Tito'));
$user = User::create(array('name' => 'Tito'));
$user->is_valid(); # => false
```

### validate (custom)

泛型方法允许自定义业务逻辑或先验证。你可以添加你自己的错误信息和错误对象。这不需要任何参数。你将逻辑放置于一个名为validate的公共方法中即可（这个特性从1.1或者测试版本中有效）。

```php
class User extends ActiveRecord\Model
{
   public function validate() 
   {
     if ($this->first_name == $this->last_name)
     {
       $this->errors->add('first_name', "can't be the same as Last Name");
       $this->errors->add('last_name', "can't be the same as First Name");
     }
   }
}
 
$user = User::create(array('first_name' => 'Tito', 'last_name' => 'Tito'));
$user->is_valid(); # => false
echo $user->errors->on('first_name'); # => can't be the same as Last Name
```

