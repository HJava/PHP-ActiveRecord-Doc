# 基础增删查改

CRUD在维基百科中的定义：

创建，查找，更新，删除是四个基本的永久存储的操作函数，所有电脑软件的主要组成部分。有时候CRUD是指扩展的文字检索而不是查找，破坏而不是删除。它有时候也用来描述用户默认接口用来帮助查找，修改信息；常用于基于电脑的表单和报告。

换句话说，CRUD是日常中常见的保存和读取数据。ActiveRecord移除和修正了冗余的难以拼写的SQL查询语句任务，你只需要编写相关的部分来操作你的数据。

## 创建

向数据库中保存数据。在这里我们创建一个新的对象请求，然后调用save（）方法。

```php
$post = new Post();
$post->title = 'My first blog post!!';
$post->author_id = 5;
$post->save();
# INSERT INTO `posts` (title,author_id) VALUES('My first blog post!!', 5)

# the below methods accomplish the same thing

$attributes = array('title' => 'My first blog post!!', 'author_id' => 5);
$post = new Post($attributes);
$post->save();
# same sql as above

$post = Post::create($attributes);
# same sql as above
```

## 读取

下面是一些基础的查找和检索数据库的方法。在查找这一节中将会有更多详细信息。

```php
$post = Post::find(1);
echo $post->title; # 'My first blog post!!'
echo $post->author_id; # 5

# also the same since it is the first record in the db
$post = Post::first();

# using dynamic finders
$post = Post::find_by_name('The Decider');
$post = Post::find_by_name_and_id('The Bridge Builder',100);
$post = Post::find_by_name_or_id('The Bridge Builder',100);

# using some conditions
$posts = Post::find('all',array('conditions' => array('name=?','The Bridge Builder')));
```

## 更新

在更新之前，你需要找到一条记录，然后修改其中的某个属性。它使用包含属性的”脏”数组（已经被修改的），所以我们的语句只更新被修改的部分。

```php
$post = Post::find(1);
echo $post->title; # 'My first blog post!!'
$post->title = 'Some real title';
$post->save();
# UPDATE `posts` SET title='Some real title' WHERE id=1

$post->update_attributes(array('title' => 'Some other title', 'author_id' => 1));
# UPDATE `posts` SET title='Some other title', author_id=1 WHERE id=1
```

## 删除

删除一条记录不会破坏这个对象。它意味着回告诉数据库删掉这条记录。然而，我们仍然还能够使用这个对象。

```php
$post = Post::find(1);
$post->delete();
# DELETE FROM `posts` WHERE id=1

echo $post->title; # Some other title
```

## 大量更新或删除

你可以轻易地更新或者删除大量数据。看下面的例子：

```php
# MASSIVE UPDATE
# Model::table()->update(AttributesToUpdate, WhereToUpdate);
Post::table()->update(array('title' => 'Massive title!', /* Other attributes... */, array('id' => array(1, 3, 7));
# UPDATE `posts` SET title = `Massive title!` WHERE id IN (1, 3, 7)

# MASSIVE DELETE
# Model::table()->delete(WhereToDelete);
Post::table()->delete(array('id' => array(5, 9, 26, 30));
# DELETE FROM `posts` WHERE id IN (5, 9, 26, 30)
```

