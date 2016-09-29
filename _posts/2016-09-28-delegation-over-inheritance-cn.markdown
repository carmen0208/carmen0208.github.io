---
layout: post
title:  "组合优于继承"
date:   2016-9-28 19:56:03 +0800
categories: Chinese
tags: ruby programming Chinese
categories: Chinese
---
## 让我们思考下边一个问题:

如果你想要实现一个和ruby本身有的类近似的数据结构，你能想到的最直接的办法可能是些一个该类的子类。
比如，如果你的数据结构近似于Array，你可以写一个类来继承Array。

所以，让我们先看一个继承的例子：

#### 继承(子类)

```ruby
class CustomList <Array
  def to_s
    join(" | ")
  end
end
```

好啦，很简单。我们也可以用它啦，而且好像看似我们可以用它做各种事情：

```ruby
cl1 = CustomList.new(%w[macbook ipad])
cl2 = CustomList.new(%w[iphone macbookpro]) 
p cl1.to_s  #=> "macbook | ipad"
p cl2.to_s  #=> "iphone | macbookpro"
cl3 =  cl1+cl2
```
直到有一天，你用了一个方法，发现他的实现和直觉想的不一样：

```ruby
p cl3.to_s  #=> "[\"macbook\", \"ipad\", \"iphone\", \"macbookpro\"]"
```

当你自能接下来仔细研究的时候，你会发现。
```ruby
p cl3.class #=> Array
```

原来当你调用+方法的时候，他调整回了Array，而不是我们自觉里边觉得的CustomList

很不爽吧，有没有一种被父类欺骗了的感觉？？？？好吧，如果有，我们提供另外一种解决方法，

## 代理（组合）

```ruby
class CustomList
  include Enumerable
  def initialize(*args)
	@list = Array.new(*args)
  end

  def to_s
	list.join(" ")
  end

  def +(other)
	self.class.new(list + other.list)
  end
 
  def each(*args, &block)
    list.each(*args, &block)
  end
  protected

  attr_reader :list
end
tl1 = CustomList.new(%w[macbook pad])
p tl1.to_s #=> "macbook | pad"
tl2 = CustomList.new(%w[iphone macbookpro])
p tl2..to_s #=> "iphone | macbookpro"
p (tl1 + tl2).to_s # => apple peach pear banana 
#include enumberable以后，所有关于列举的功能也都附带实现了。比如：
tl3.grep(/p/) 
tl3.map(&:reverse)
tl3.group_by(&:size)
```

所以，这里带给我们的观点是：当我们考虑实现一些已提供的类的近似功能，为了不被他欺负或欺骗，
我们可以选择组合而不是继承（更不用提，通常我们只能继承至一处，但是我们可以组合世间万物
：））

## 那么这段代码教会了我什么样的人生道理呢?

在我们的生活中，通常我们可以从祖辈身上继承太多的东西，有时候我们甚至有种可以不劳而获的感觉。
可是，在你继承这样的东西的时候，可能他们也会带走你身上的某种特质。让自己动起来，找到更多的人组合起来
来帮助你，可能比直接的继承更加有力量，更加实际和真实！

#### 附带说一句,

上边的方法，如果你还是想继承，那当遇到上溯问题的时候，你也可以考虑过勤劳一点，实现一些方法
来让你减少一些权力的被剥夺感 ：）

实例方法如下：

```ruby

class CustomList <Array
  def to_s
    join(" | ")
  end

  def +(other)
    mid = super(other)
    self.class.new(mid)
  end
end

```

