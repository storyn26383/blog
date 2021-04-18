---
title: 設計模式 - 偷懶工廠模式
tags:
  - 設計模式
  - PHP
---

工廠模式我想大家一定不陌生，但偷懶工廠又是什麼呢？
我們會想要用工廠模式，基本上就是為了要解耦合，但付出的代價似乎不小，不僅造成日後擴充的問題，還大大增加了程式複雜度。
那我們有沒有什麼辦法可以用工廠模式來解耦合，同時又可以擴充容易，也不會讓程式複雜度增加太多呢？
這時偷懶工廠模式就登埸了！

<!--more-->

## 簡單工廠模式

首先讓我們複習一下簡單工廠模式：

```php
class DatabaseClientFactory
{
    public static create(string $driver)
    {
        if ($driver === 'MySQL') {
            return new MySQLClient;
        }

        if ($driver === 'SQLite') {
            return new SQLiteClient;
        }

        throw new Exception("{$driver} is not supported.");
    }
}

$mysqlClient = DatabaseClientFactory::create('MySQL');
$sqliteClient = DatabaseClientFactory::create('SQLite');
```

簡單工廠最大的缺點就是：擴充時需要修改 `DatabaseClientFactory`，違反了開放封閉原則。

## 工廠模式

為了要符合開放封閉原則，因此我們改用工廠模式：

```php
class MySQLClientFactory
{
    public static create()
    {
        return new MySQLClient;
    }
}

class SQLiteClientFactory
{
    public static create()
    {
        return new SQLiteClient;
    }
}

$mysqlClient = MySQLClientFactory::create();
$sqliteClient = SQLiteClientFactory::create();
```

雖然符合開放封閉原則了，擴充變得更加容易，但程式複雜度也增加了，難到不能兩全齊美嗎？
答案是可以的！讓我們繼續看下去～

## 偷懶工廠模式

廢話不多說，直接看程式碼：

```php
class DatabaseClientFactory
{
    public static create(string $driver)
    {
        $className = "{$driver}Client";

        if (! class_exists($className)) {
            throw new Exception("{$driver} is not supported.");
        }

        return new $className;
    }
}

$mysqlClient = DatabaseClientFactory::create('MySQL');
$sqliteClient = DatabaseClientFactory::create('SQLite');
```

主要是利用 PHP 的語言特性，讓我們可以動態組成 Class Name，如此一來不僅讓程式變得更簡單，還同時符合了開放封閉原則，還真是兩全齊美呢！
但，真的是這樣嗎？其實這麼做還是有缺點的，那就是無法簡單追蹤到 Class 被實例化的地方，造成程式碼維護不易，也因此我把他稱為偷懶工廠模式，沒錯，這個名字是我自己取的 XD

## 結語

沒有十全十美的設計模式，但你一定可以找到最適合當下的設計模式。
