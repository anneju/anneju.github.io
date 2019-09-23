---
layout: post
title: Ruby 世界裡的符號（Symbol）是什麼？它跟字串又有什麼不同？
date: 2019-09-22
Author: Anne Ju
tags: [Ruby, Symbol]
comments: true
toc: true
pinned: true
---

對於Ruby的新手，看到「符號」這個東西實在很難理解。今天試著來解釋一下到底是什麼。

在Ruby裡會看到 `:name `、 `:success` 等前面有冒號的字串，這個就是Ruby裡的符號(Symbol)。

**Symbol不是變數，他自己就是個「值」。** 我們打開irb看看：

```ruby
2.6.3 :024 > a = 1
 => 1 
＃數字物件

2.6.3 :025 > a = "hello"
 => "hello" 
＃字串物件

2.6.3 :026 > a = :hello
 => :hello
＃符號物件

＃我們可以把一個變數設為數字、字串、或是symbol
2.6.3 :027 > :hello = a
SyntaxError ((irb):27: syntax error, unexpected '=', expecting end-of-input)
:hello = a
＃但當我們把symbol設為變數時，就出現SyntaxError
```

其實它就是個「有名字的物件(an object with a name)」，某種意義上可以理解為「無法變更的字串(String)」。

我們來看看它和字串(String)有什麼不一樣。

# **字串的內容可以被修改，但 Symbol 不行**

再次打開irb，我們試著用字串的方法來套用到symbol上：

```ruby
2.6.3 :010 > "hello".length
 => 5 #得到5
2.6.3 :011 > :hello.length
 => 5 #得到5
2.6.3 :012 > "hello"[0]
 => "h" #得到第一個字h
2.6.3 :013 > :hello[0]
 => "h" #得到第一個字h
```

但如果我們今天試著來改變其內容

```ruby
2.6.3 :016 > "hello"[0] = "A"
 => "A"
2.6.3 :017 > :hello[0] = "A"
NoMethodError (undefined method `[]=' for :hello:Symbol)
```

會發現他無法被改變，所以symbol非常適合拿來作為不能被修改的字。當我們要把symbol轉為字串時，可以用.to_s

另外，每個symbol產生時，都會在Ruby裡面建立單一的object id，而每個字串在產生時，都會生成一個新的object id。因此，字串的效能比較不好，也比較佔記憶體空間。

```ruby
2.6.3 :018 > puts "hello".object_id
70254198830520
 => nil
2.6.3 :019 > puts "hello".object_id
70254198792800
 => nil
2.6.3 :020 > puts "hello".object_id
70254210379160
=> nil
＃字串每次的object id都會不一樣
2.6.3 :021 > puts :hello.object_id
1524508
 => nil
2.6.3 :022 > puts :hello.object_id
1524508
 => nil
2.6.3 :023 > puts :hello.object_id
1524508
 => nil
＃symbol每次的object id都是固定的
```

正因為這樣的特性，symbol也很適合拿來當作hash的key，因為指向記憶體的同一個位置。
