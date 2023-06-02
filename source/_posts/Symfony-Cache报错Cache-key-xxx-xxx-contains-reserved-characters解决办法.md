---
title: Symfony Cache报错Cache key 'xxx:xxx' contains reserved characters解决办法
date: 2023-06-02 14:57:04
tags: ["PHP", "Symfony"]
categories: ["技术"]
---

今天在使用Symfony Cache时遇到一个报错：`Symfony\Component\Cache\Exception\InvalidArgumentException: Cache key "xxx:xxx" contains reserved characters "{}()/\@:"`，经过排查发现我的缓存key带有冒号。
<!-- more -->
根据报错信息找源头，文件`vendor/symfony/cache/Traits/AbstractAdapterTrait.php`第396行有如下代码：

```PHP
\assert('' !== CacheItem::validateKey($key));
```

这里有一个断言，会根据`CacheItem::RESERVED_CHARACTERS`(值为`{}()/\@:`)来判断缓存key是否合法。

通过PHP官方文档对[assert](https://www.php.net/manual/zh/function.assert.php)的说明，发现PHP7以上版本有如下配置：

|  指令  | 默认值  |  可能值  |
|  ----  | ----  | ----  |
| [zend.assertions](https://www.php.net/manual/zh/ini.core.php#ini.zend.assertions)  | 1 | 1：生成并执行代码（开发模式）<br>0：生成代码但在运行时跳转<br>-1：生成代码但在运行时跳转（生产模式） |
| [assert.exception](https://www.php.net/manual/zh/info.configuration.php#ini.assert.exception)  | 0 | 1：断言失败时抛出，抛出提供给 exception 的对象，或未提供 exception 时抛出新的 AssertionError 对象<br>0：如上所述使用/生成 Throwable，但仅仅是基于该对象生成警告而不是抛出（与 PHP 5 行为兼容） |

所以解决办法有两种：

 - 修改`php.ini`配置，将`zend.assertions`的值设置为0或-1

 - 去除缓存key中的特殊符号
