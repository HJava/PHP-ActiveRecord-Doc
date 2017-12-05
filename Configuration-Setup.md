# 配置/设置

设置非常简单直白。本质上来讲，这里只有两个设置点需要你关注：

1. 配置自动导入模型的目录
2. 配置你的数据库连接

通过设置模型自动导入的目录，你能够告诉PHP从何处去寻找你的模型类。这意味着你能够有一个你选择的应用程序/文件夹结构，只要你有一个真正的目录保存你的模型类。每一个类应该有一个单独的php文件，文件名应该与类名的扩展相同。

现在有两种方式来初始化你的配置选项。最简单的方法是封装这些调用到一个闭包中，传递给配置的initializer方法调用。下面是一个最优雅简洁的方法，它充分利用了PHP新闭包特性的优点。

```php
# inclue the ActiveRecord library
require_once 'php-activerecord/ActiveRecord.php';

ActiveRecord\Config::initialize(function($cfg)
{
  $cfg->set_model_directory('/path/to/your/model_directory');
  $cfg->set_connections(array('development' =>
    'mysql://username:password@localhost/database_name'));
});
```

就是这样。ActiveRecord会帮你完成其他配置。你不需要自己使用yaml/xml来映射表模型。它会自己查询数据库来获取这些信息并缓存下来，同时不会产生对一个单模型数据库产生多重的请求。

如果你不想这样，你可以抛弃闭包，通过ActiveRecord\Config来直接使用。

```php
$cfg = ActiveRecord\Config::instance();
$cfg->set_model_directory('/path/to/your/model_directory');
$cfg->set_connections(array('development' =>
  'mysql://username:password@localhost/database_name'));
```

## 默认连接

开发连接按照默认值。你能够设置一个基于你传递的某个连接来修改它的默认连接。

```php
$connections = array(
  'development' => 'mysql://username:password@localhost/development',
  'production' => 'mysql://username:password@localhost/production',
  'test' => 'mysql://username:password@localhost/test'
);

# must issue a "use" statement in your closure if passing variables
ActiveRecord\Config::initialize(function($cfg) use ($connections)
{
  $cfg->set_model_directory('/path/to/your/model_directory');
  $cfg->set_connections($connections);

  # default connection is now production
  $cfg->set_default_connection('production');
});
```

## 多连接

你能够很简单的配置ActiveRecord来接受多个数据库的连接。你需要做的仅仅是指定你所给的用于不同数据库的模型来连接。

```php
$connections = array(
  'development' => 'mysql://username:password@localhost/development',
  'pgsql' => 'pgsql://username:password@localhost/development',
  'sqlite' => 'sqlite://my_database.db',
  'oci' => 'oci://username:passsword@localhost/xe'
);
 
# must issue a "use" statement in your closure if passing variables
ActiveRecord\Config::initialize(function($cfg) use ($connections)
{
  $cfg->set_model_directory('/path/to/your/model_directory');
  $cfg->set_connections($connections);
});
```

你的模型应该看上去如下。

```php
# SomeOciModel.php
class SomeOciModel extends ActiveRecord\Model
{
  static $connection = 'oci';
}

# SomeSqliteModel.php
class SomeSqliteModel extends ActiveRecord\Model
{
  static $connection = 'sqlite';
}
```

你还应该有一个基础连接模型，这样，所有的子类都会继承那只的数据库设置。

```php
# OciModels.php
abstract class OciModels extends ActiveRecord\Model
{
  static $connection = 'oci';
}

# AnotherOciModel.php
class AnotherOciModel extends OciModels
{
  # automatically inherits the oci database
}
```

## 设置编码

字符编码可以在你连接的参数中指定：

```php
$config->set_connections(array(
  'development' => 'mysql://user:pass@localhost/mydb?charset=utf8')
);

```

