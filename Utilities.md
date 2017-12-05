# 实用程序

ActiveRecord提供了许多的方法来向我们的模型中添加一些有趣的特性从而使其变得更易使用。

## 代表

除了这个是通过你的关联来工作，其他与属性别名很类似。你可以为你的模型中的某个属性起一个别名来在一个关联中使用这个特殊的属性。让我们来看看。

```php
class Person extends ActiveRecord\Model {
   static $belongs_to = array(array('venue'),array('host'));
   static $delegate = array(
     array('name', 'state', 'to' => 'venue'),
     array('name', 'to' => 'host', 'prefix' => 'host'));
 }
 
 $person = Person::first();
 $person->state     # same as calling $person->venue->state
 $person->name      # same as calling $person->venue->name
 $person->host_name # same as calling $person->host->name
```

## 属性设置器（setters）

设置器运行你定义常规的方法来指定你某个属性中的值。这意味着你可以拦截执行的过程来过滤或者修改你的数据。这在类似加密密码的场合很有用。通常，通常,您定义一个设置器不携带相同的名称作为你的属性,但是你可以设置属性的方法。在下面的例子中，$user->password是虚拟的属性：如果对这个属性你尝试读取而不是赋值，那么UndefinedPropertyException错误将会被抛出。

```php
class User extends ActiveRecord\Model {
  # A setter method must have set_ prepended to its name to qualify.
  # $this->encrypted_password is the actual attribute for this model.
  public function set_password($plaintext) {
    $this->encrypted_password = md5($plaintext);
  }
}

$user = new User;
$user->password = 'plaintext';  # will call $user->set_password('plaintext')
# if you did an echo $user->password you would get an UndefinedPropertyException
```

如果你定义了一个属性与常规的设置器同名，那么你需要使用assign_attribute方法来对你的属性赋值。这需要取决于Model：：__set方法的运行。例如：假定‘name’是表中的某一列，然后你定义了一个常规的设置器也叫做‘name’。

```php
class User extends ActiveRecord\Model {

  # INCORRECT:
  # function set_name($name) {
  #   $this->name = strtoupper($name);
  # }

  public function set_name($name) {
    $this->assign_attribute('name',strtoupper($name));
  }
}

$user = new User;
$user->name = 'bob';
echo $user->name; # => BOB
```

## 属性获取器（getters）

获取器允许你拦截你模型中的数据属性检索。他们与设置器定义相似。请于Model：：__get来查看详情。

属性别名

这是一个选项是相当直接的。一个属性别名允许你通过一个不同的名字来设置/获取属性。这能在你有糟糕的列名像列一，列二或者遗留表中起到作用。在这个例子中，first_name的别名person_first_name是被已经存在的列引用的。

```php
class Person extends ActiveRecord\Model {
  static $alias_attribute = array(
    'first_name' => 'person_first_name',
    'last_name' => 'person_last_name');
}

$person = Person::first();
echo $person->person_first_name; # => Jax

$person->first_name = 'Tito';
echo $person->first_name; # => Tito
echo $person->person_first_name; # => Tito
```

## 受限制的属性值

黑名单属性是不能被指定的。保护这些属性运行能够让你避免例如恶意用户会创建一些特殊的请求值等问题。与其相对的是可使用属性。

```PHP
class User extends ActiveRecord\Model {
  static $attr_protected = array('admin');
}

$attributes = array('first_name' => 'Tito','admin' => 1);
$user = new User($attributes);

echo $user->first_name; # => Tito
echo $user->admin; # => null
# now no one can fake post values and make themselves an admin against your will!
```

## 可使用属性

白名单属性是从指定中检查例如创建一个模块使用Model：：update_attributes()。
与其相对的是受限制属性。可使用属性也能够被用来作为安全措施避免指定假冒请求值，而且因为这是一个白名单方法所以更加常用的多。

```php
class User extends ActiveRecord\Model {
  static $attr_accessible = array('first_name');
}

$attributes = array('first_name' => 'Tito','last_name' => 'J.','admin' => 1);
$user = new User($attributes);

echo $person->last_name; # => null
echo $person->admin; # => null
echo $person->first_name; # => Tito
# first_name is the only attribute that can be mass-assigned, so the other 2 are null
```

## 序列化

这不是你使用的通常类型的PHP序列化。它不会序列化你的整个对象，然而，他会序列化你的模型属性为一个XML或者json类型。一个选项数组可以使用以下的参数：

- only：一个被包含的属性字符串或者数组。
- except:一个被排除的属性字符串或者数组。
- methods：一个调用方法所使用的字符串或者数组。这方方法名将会被作为最终属性数组和方法的返回值的键值。
- include：关联模型的字符串或者数组，包括最终序列化的结果。
- skip_instruct：设置跳过声明。

下面仅仅是包含Model：：to_json的方法的例子。然而，你可以在所有例子中使用model：：to_xml方法。

```php
class User extends ActiveRecord\Model {
   static $has_many = array(array('orders'));
 
   public function name() {
     return $this->first_name .' '. $this->last_name;
   }
}

# assume these fields are on our `users` table:
# id, first_name, last_name, email, social_security, phone_number

$user = User::first();

# json should only contain id and email
$json = $user->to_json(array(
  'only' => array('id', 'email')
));
 
echo $json; # => {"id":1,"email":"none@email.com"}
 
# limit via exclusion (here we use a string, but an array can be passed)
$json = $user->to_json(array(
   'except' => 'social_security'
));
 
echo $json; # => {"id":1,"first_name":"George","last_name":"Bush",
             #     "email":"none@email.com","phone_number":"555-5555"}
 
# call $user->name() and the returned value will be in our json
$json = $user->to_json(array(
   'only' => array('email', 'name'),
   'methods' => 'name'
));
 
echo $json; # => {"name":"George Bush","email":"none@email.com"}
 
# include the orders association
$json = $user->to_json(array(
   'include' => array('orders')
));
 
# you can nest includes .. here orders also has a payments association
$json = $user->to_json(array(
   'include' => array('orders' => array('except' => 'id', 'include' => 'payments')
));
```

DateTime列被默认序列化成了ISO8601标准格式。这个格式能够被改成ActiveRecord\Serialization::$DATETIME_FORMAT。你可以使用在DateTime::$FORMAT中提前定义的任何格式化方法。

```php
ActiveRecord\Serialization::$DATETIME_FORMAT= 'Y-m-d';
ActiveRecord\Serialization::$DATETIME_FORMAT= 'atom';
ActiveRecord\Serialization::$DATETIME_FORMAT= 'long';
ActiveRecord\Serialization::$DATETIME_FORMAT= \DateTime::RSS;
```

## 自动时间戳

模型中有列名被称为create_at和updated_at的将会根据模型创建和更新自动更新这些列。

