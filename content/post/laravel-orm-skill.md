---
title: "Laravel ORM 中那些你不知道的骚操作"
date: 2020-04-01
lastmod: 2020-04-01
draft: false
tags: ["Laravel", "ORM"]
categories: ["PHP", "Laravel"]
author: "zxdstyle"
contentCopyright: '本站所有文章无特殊说明都是原创，转载请注明出处<a rel="license noopener" href="https://www.zxdstyle.cn" target="_blank">www.zxdstyle.cn</a>'

---

# append
```php
class User extends Model
{
    protected $appends = ['is_adult'];
    
    public function getIsAdultAttribute()
    {
        return $this->attribute['age'] > 18;
    }
}
```
这个操作大家是不是都用过，在模型里新增一个数据库不存在的字段，非常方便。但是 $appends 是全局的，所有的查询中都会添加 is_adult 这个字段。
```php
User::select('id', 'name')->first();
```
像这样查询的时候甚至还会报错提示 age 字段不存在。

我们可以像这样，在查询的时候再将 is_adult 添加进查询结果集中。
```php
$user = User::first();

$user->append('is_adult');
```

你以为这就完了么？不仅仅如此，如果我们查询的是多个用户怎么办，难道自己 for 循环 append 一遍么？不不不，我们优雅的 Laravel 已经为我们考虑过了。
```php
$user = User::paginate(10);

$user->each->append('is_adult');
```

# query
```php
User::where('sex', 'girl')->where('age', '<=', 20)->where('money', '>', 1000000000000)->get();
```
这种查询语句大家是不是经常写啊？有没有发现一个问题？本来找个富萝莉就挺难得，还没有提示。

![Laravel ORM 中你不知道的骚操作](https://cdn.learnku.com/uploads/images/202004/01/39723/PN7TBaOI3M.png!large)

这怎么能忍，稍稍改写一下，在最前面加个 `query` ，轻轻松松娶富萝莉走上人生巅峰。

![Laravel ORM 中你不知道的骚操作](https://cdn.learnku.com/uploads/images/202004/01/39723/homlqOvgw1.png!large)

# where
富萝莉没找到的话，降低点要求正儿八经找个女朋友吧。虽然有点难，但是如果你知道她的 ID，就可以直接使用
```php
User::query()->find(2);
```
找到她，简单快捷。那要是不知道 ID 只知道名字的情况下咋整呢？写where条件？告诉你个更快捷的方法，毕竟找女朋友不能等。
```php
User::query()->firstWhere(['name' => '乔碧萝']);
```


先写这么多，发现其他骚操作再更:smirk:。
