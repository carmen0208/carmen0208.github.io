---
layout: post
title:  "Delegation over Inheritance"
date:   2016-9-28 19:26:03 +0800
categories: English
tags: ruby programming English

---
#### Let's think about this:

If you want a data structure that almost the same as ruby Array but different in
some part. You might make an subclass of Array like the example below:

###### Inheritance(Subclass)

```ruby
class CustomList <Array
  def to_s
    join(" | ")
  end
end
```

then when you play with it, everthings is fine...

```ruby
cl1 = CustomList.new(%w[macbook ipad])
cl2 = CustomList.new(%w[iphone macbookpro]) 
p cl1.to_s  #=> "macbook | ipad"
p cl2.to_s  #=> "iphone | macbookpro"
cl3 =  cl1+cl2
```
util, you might encounter the problem:

```ruby
p cl3.to_s  #=> "[\"macbook\", \"ipad\", \"iphone\", \"macbookpro\"]"
```

and when you dig into it, here's the reason:
```ruby
p cl3.class #=> Array
```

it changed back to **Array** instead of **CustomList**.......and that's counter intuition

##So Here's an other way-- Delegation:

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
#any enumberable function it could be easily implemented.
tl3.grep(/p/) 
tl3.map(&:reverse)
tl3.group_by(&:size)
```

So here's an idea: when considerring implement sometime like the component that already have, 
it's better to use delegation rather that inheritance.

##What's the code told?

It's common in our life that you can inheritance almost everything from your parents, but it would take
your specific identity away as well if you rely them much, it's better to find someone help 
out side rather than directly require everything from parents
