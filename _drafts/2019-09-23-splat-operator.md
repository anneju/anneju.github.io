---
layout: post
title: Ruby 世界裡的符號（Symbol）是什麼？它跟字串又有什麼不同？
date: 2019-09-23
Author: Anne Ju
tags: [Ruby, splat]
comments: true
toc: true
pinned: true
---

# Ruby參數中的星號*splat operator (*)

先講總結：

- Ruby參數(paramater)中的星號*可以讓我們在不確定會有多少引數(arguments)時使用
- Ruby參數(paramater)中的星號*只會代入沒有相對應位置的引數(arguments)
- Ruby參數(paramater)中的星號*是選擇性(optional)使用的，即使不傳入引數也沒關係
- 如果沒有相應的引數(arguments)，Ruby參數(paramater)中的星號*則直接產生空的陣列
- Ruby參數(paramater)中的星號*會將代入的引數(arguments)轉為陣列(Array)
- Ruby參數(paramater)中的星號*產生的陣列順序依照引數代入順序
- 一個function只能有一個星號*參數

# **Ruby參數(paramater)中的星號\*可以讓我們在不確定會有多少引數(arguments)時使用**

假設我們今天要做一個收銀機，發票上要列出客人購買的品項有多少。但我們不確定客人會買哪些東西，又會買多少項。

```ruby
def total_items(*item)  
  item.count
end

p total_items("water") 
p total_items("water", "cookie")
p total_items("water", "cookie", "hotdog")
```

結果如下

```
1
2
3
```

由此可見*會幫我們做這個計算。我們不需要計算引數，只要把字串帶入。由此可知，在我們不確定會有多少引數時，可以利用*的特性幫助我們。

------

# **Ruby參數(paramater)中的星號\*只會代入沒有相對應位置的引數(arguments)**

例如，如果我們已經有一個確定會帶入引數的參數，後面跟個有加上*的參數參數：

（下面舉例，如果我要列出一個團隊出遊的名單，目前只確認導遊的名字（因為導遊一定會參加，但還不確定其他團客有誰），可以這樣寫）

```ruby
def tour_participants(guide, *guests)  
  "The tour guide is #{guide}. And the guests are #{guests}."
end

print tour_participants("Mary", "Tom", "Carlos", "Anne", "Winny")
```

結果如下：

```
The tour guide is Mary. And the guests are ["Tom", "Carlos", "Anne", "Winny"].
```

由此可見，*參數會幫我們把沒有地方放的引數帶入。

------

# Ruby參數(paramater)中的星號*是選擇性(optional)使用的，即使不傳入引數也沒關係

如果我們今天在function中設定了有*的參數，但即使不傳入值，Ruby也不會回傳錯誤。

以前面的例子來說：

```ruby
def tour_participants(guide, *guests)  
  "The tour guide is #{guide}. And the guests are #{guests}."
end

puts tour_participants("Mary")
```

得到結果：

```
The tour guide is Mary. And the guests are [].
```

但是如果我們連第一個參數都沒有帶入引數，就會發生錯誤。

------

# 如果沒有相應的引數(arguments)，Ruby參數(paramater)中的星號*則直接產生空的陣列

再次看上面的例子，我們稍微修改一下：

```ruby
def tour_participants(guide, *guests)  
  guests.inspect
end

puts tour_participants("Mary")
```

得到結果：

```ruby
[]
```

------

# Ruby參數(paramater)中的星號*會將代入的引數(arguments)轉為陣列(Array)

我們從上面的結果已經知道*會將所帶入的引數轉為陣列，我們也可以拿這陣列作運算。

```ruby
def add_all_args (num1, num2, *num)  
  p num.class  sum = num1 + num2  
  num.each { |n| sum += n}  sum
end

puts add_all_args(1,2,3,4,5,6,7,8,9,10)
```

得到結果：

```
Array
55
```

我們也可以由第二行的p num.class得知此為一個Array。

------

# Ruby參數(paramater)中的星號*產生的陣列順序依照引數代入順序

利用*參數帶入得引數會依照帶入的順序傳入function中。

```ruby
def print_pramas(*word)  
	word
end

p print_pramas(1, 2, 3, 4, 5)
```

得到結果：

```
[1, 2, 3, 4, 5]
```

------

# 一個function只能有一個星號*參數

讓我們試著使用兩個有*的參數：

```ruby
def shopping_list(*food, *grocery)  
  p food  
  p grocery
end
  
p hopping_list("Bread", "Milk", "Flower", "Toliet paper")
```

得到錯誤訊息：

```ruby
rb:1: syntax error, unexpected *
def shopping_list(*food, *grocery)
rb:4: syntax error, unexpected end, expecting end-of-input
```

本篇參考[Parameter with splat operator (*) in Ruby](https://medium.com/@sologoubalex/parameter-with-splat-operator-in-ruby-part-1-2-a1c2176215a5)這篇文章