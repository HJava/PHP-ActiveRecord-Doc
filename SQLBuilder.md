# SQL构建器

这个指南将告诉你如何使用php-activerecord构建器。

第一步是将你的数据库练剑并且设置一个sql构建器。

```php
$conn = ActiveRecord\ConnectionManager::get_connection("development");                                                                                                                         
$builder = new ActiveRecord\SQLBuilder($conn, "authors"); 
```

## 选择查询

我们现在准备构建一个简单的SELECT语句。

```php
$builder->where("name = ?", "Hemingway");
echo $builder; /* => SELECT * FROM authors WHERE name = ? */
print_r($builder->get_where_values()); /* => array("Hemingway") */
```

你也可以传递一个拼凑值给where方法。

```php
$builder = new ActiveRecord\SQLBuilder($conn, "authors");                                                                                                            
$builder->where(array("name" => "Hemingway",                                                                                                                         
                       "country" => "USA"));
echo $builder; /* => SELECT * FROM authors WHERE `name`=? AND `country`=? */
print_r($builder->get_where_values()); /* => array('Hemingway', 'USA'); */
```

您可以添加订购信息。

```php
$builder = new ActiveRecord\SQLBuilder($conn, "authors");
$builder->order('id DESC');
echo $builder."\n"; /* => SELECT * FROM authors ORDER BY id DESC */
```

酷~