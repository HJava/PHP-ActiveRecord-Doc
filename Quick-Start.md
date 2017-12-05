# 快速入门

这个入门介绍将会向你展示最基本的php-activerecord的配置与运行。我假设你已经下载库文件到你的名为php-activerecord的包含路径中。

第一步是引入这个库并且定义与数据库的连接：

```php
require_once 'php-activerecord/ActiveRecord.php';
ActiveRecord\Config::initialize(function($cfg) {
    $cfg->set_model_directory('models');
    $cfg->set_connections(array(
        'development' => 'mysql://username:password@localhost/database_name'));
});
```

接下来，让我们创建一张叫做users的表。我们在名为models/User.php的文件中包含这个class：

```php
class User extends ActiveRecord\Model {}
```

就是这样。现在我们能够通过User模型来使用users表：

```php
# create Tito
$user = User::create(array('name' => 'Tito', 'state' => 'VA'));

# read Tito
$user = User::find_by_name('Tito');

# update Tito
$user->name = 'Tito Jr';
$user->save();

# delete Tito
$user->delete();
```

就是这样。非常简单。通过查看框架部分来获取更多的在特殊框架中的使用php-activerecord的深度指南。