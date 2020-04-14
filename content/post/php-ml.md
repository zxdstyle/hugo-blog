---
title: "PHP 实现机器学习挖掘用户的购物习惯"
date: 2020-03-13
lastmod: 2020-03-13
draft: false
tags: ["机器学习", "PHP"]
categories: ["PHP"]
author: "zxdstyle"
contentCopyright: '本站所有文章无特殊说明都是原创，转载请注明出处<a rel="license noopener" href="https://www.zxdstyle.cn" target="_blank">www.zxdstyle.cn</a>'

---

# 开场白

经常听说某某某专攻机器学习的程序猿的年薪百万,AI 更是火得不得了:smirk:，提起这些基本都是 Python 、Java、Go、甚至 JavaScript 都有就是没有我们 PHP 什么事:unamused:，这我就不服我了啊。毕竟
> PHP 是世界上最好的语言:laughing:

虽然 PHP 的生态导致无法落地机器学习，但还是可以做一些简单的数据分析工作的。

# 安装

这里推荐 `php-ml` 这个库，其中包含各种算法，交叉验证，神经网络，预处理，特征提取等等。需要 PHP 版本 `7.1` 以上， 速度还是可以的。
```bash
	composer require php-ai/php-ml
```
# Apriori 算法

> Apriori算法是一种挖掘关联规则的频繁项集算法

说简单点，Apriori 算法就是在已有的数据里根据出现的组合频次来总结一定的关联关系。

这个详细算法不去深究，简单介绍一下他的两个参数。

## 支持度[Support]

$$\displaystyle Support=\frac{同时购买「香烟、打火机」的人数}{总人数}$$

很好理解，20个购买香烟的人里有19个人都买了打火机，那香烟和打火机肯定是有一定关联。


## 置信度[Confidence ]

置信度就是购买 a 的人同时购买 b 的概率。例如：

（香烟 => 打火机）的置信度：

$$\displaystyle confidence =\frac{同时购买香烟和打火机的人数}{购买香烟的人数}$$

（打火机 => 香烟）的置信度：

$$\displaystyle confidence =\frac{同时购买香烟和打火机的人数}{购买打火机的人数}$$

# 数据训练

理解这两个参数之后我们开始训练我们的机器，让他知道用户购买了香烟之后应该给用户推荐什么。

首先准备好样本数据：
```php
$samples = [

    ['香烟','打火机'],

    ['香烟','炸鸡','啤酒','鸡排'],

    ['打火机','炸鸡','啤酒','可乐'],

    ['香烟','打火机','炸鸡','啤酒'],

    ['香烟','打火机','炸鸡','可乐']

];
```

根据上面的公式计算出支持度和可信度。然后开始训练：
```php
use Phpml\Association\Apriori;

$associator  =  new  Apriori($support  =  0.5,  $confidence  =  0.5);

$associator->train($samples,  []);
```
速度还是可以的。训练完成之后我们可以获取生成的关联规则：
```php
$associator->getRules()
```
然后我们可以来预测一下买了打火机之后，用户可能会买什么：
```php
$associator->predict(['打火机'])

// [
//	0 => [ '香烟' ],
//	1 => [ '炸鸡'],
// ]
```

> 现在你知道用户买完避孕套该给他推荐啥了吧:smirk:

当然，如果你想保存训练好的模型，下次再使用的话，可以：
```php
	use Phpml\ModelManager;

	// 保存训练好的模型
	$filepath = '/path/to/store/the/model'; 
	$manager = new ModelManager();
	$manager->saveToFile($associator, $filepath);
```
从文件读取训练好的模型：
```php
	// 使用
	$manager = new ModelManager();
	$filepath = '/path/to/store/the/model'; 
	$restoredAssociator = $manager->restoreFromFile($filepath);
	$restoredAssociator->predict(['炸鸡']);
```