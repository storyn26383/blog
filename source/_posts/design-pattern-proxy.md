---
title: 設計模式 - 代理模式 x 快取 x Metaprogramming
date: 2021-04-20 22:03:18
tags:
  - 設計模式
  - PHP
  - Cache
  - Metaprogramming
---

當我們想為某個 Class 裡 Method 的執行結果加上快取，直覺就會聯想到 - 代理模式。
但如果 Method 一多，代理層寫起來就會又臭又長，一大堆重覆的東西，想到就累了。
那我們有沒有什麼辦法可以簡化代理層，讓開發更省力呢？當然有！那就是 Metaprogramming。

<!--more-->

## 傳統 Proxy

我們先來看看傳統的寫法

```php
class Subject
{
    public function getFoo()
    {
        return 'foo';
    }

    public function getBar()
    {
        return 'bar';
    }
}

class Proxy
{
    private Subject $subject;
    private array $cache = [];

    public function __construct(Subject $subject)
    {
        $this->subject = $subject;
    }

    public function getFoo()
    {
        if (! isset($this->cache['getFoo'])) {
            $this->cache['getFoo'] = $this->subject->getFoo();
        }

        return $this->cache['getFoo'];
    }

    public function getBar()
    {
        if (! isset($this->cache['getBar'])) {
            $this->cache['getBar'] = $this->subject->getBar();
        }

        return $this->cache['getBar'];
    }
}
```

我們可以發現到 `Proxy` 裡有大量重覆的東西，看了實在非常不舒服 X_X

## 加入 Metaprogramming

拜 PHP 所𧶽，我們可以非常簡單實作 Metaprogramming，PHP 提供了大量的 Magic Methods，這次我們會使用到的是 `__call()`。
`__call()` 是什麼呢？來看一段小程式：

```php
class Proxy
{
    public function __call($method, $args)
    {
        var_dump($method);
    }
}

$proxy = new Proxy;
$proxy->getFoo(); // string(6) "getFoo"
```

從上面的例子可以發現，`Proxy` 中明明就沒有 `getFoo()` 這個 Method，但第 10 行的程式卻不會出錯，反而還有印出結果，為什麼呢？
這是因為 `Proxy` 有 `__call()` 這個 Method，只要呼叫了沒有宣告過的 Method，就會往 `__call()` 跑。

此時我們就可以利用這個特性來改寫我們的快取代理：

```php
class Proxy
{
    private Subject $subject;
    private array $cache = [];

    public function __construct(Subject $subject)
    {
        $this->subject = $subject;
    }

    public function __call($method)
    {
        if (! isset($this->cache[$method])) {
            $this->cache[$method] = $this->subject->$method();
        }

        return $this->cache[$method];
    }
}
```

如此一來，重覆的程式碼就都不見了呢！就算 `Subject` 裡的 Method 再多也不用怕了～
但，真的是這樣嗎？
其實這麼做還是有缺點的，`Proxy` 裡並沒有宣告被呼叫的 Method，使程式碼追蹤不易，進而造成維護上的困難。

## 結語

雖然運用 `__call()` 讓我們在開發上節省了許多力氣，但如果我們今天是團隊合作開發時，還是要三思呀！
