---
layout: post
title:  "Why Methods are not first-class object in Ruby"
date:   2016-09-25 14:26:03 +0800
tags: ruby programming
---

## What is first-class object?

Objects can not be passing through methods directly when created 

{% highlight ruby %}
def plus1 x
  x + 1
end

[1,2,3,4].map &plus1
#=>`plus1': wrong number of arguments (0 for 1) (ArgumentError)
{% endhighlight %}

But you can upgrade it to an object to makes is possible

{% highlight ruby %}
def plus1 x
  x + 1
end
first_class_plus1 = method(:plus1)
[1,2,3,4].map &first_class_plus1
#=>[2, 3, 4, 5]
{% endhighlight %}

## Why aren't Ruby methods first-class?

Because is fundamentally consistent with Ruby's nature as a purely OO language.

## But ..... Why not just upgrade it as an object and passing those methods?

**Example1**

```ruby
class Bag
  def draw(text)
    puts text
  end

end

module AdsPlugin
  def draw(*)
    puts "copyright: Ninja Company!"
    super
  end

end

class Manufactory
  attr_accessor :client, :bag
  def initialize
    bag = Bag.new
    @client = Client.new(bag.method(:draw))
    #@client = Client.new(bag)
    bag.extend(AdsPlugin)
  end

  def draw
    @client.draw
  end
end

class Client
  attr_accessor :bag
  def initialize bag
    @bag = bag
  end

  def draw
    @bag.call("I love my Bag!")
    #@bag.draw("I love my Bag!")
  end
end

d = Manufactory.new.draw
#result: 

#=>I love my Bag!
```

**Example2**

```ruby
class Bag
  def draw(text)
    puts text
  end

end

module AdsPlugin
  def draw(*)
    puts "copyright: Ninja Company!"
    super
  end

end

class Manufactory
  attr_accessor :client, :bag
  def initialize
    bag = Bag.new
    #@client = Client.new(bag.method(:draw))
    @client = Client.new(bag)
    bag.extend(AdsPlugin)
  end

  def draw
    @client.draw
  end
end

class Client
  attr_accessor :bag
  def initialize bag
    @bag = bag
  end

  def draw
    # @bag.call("I love my Bag!")
    bag.draw("I love my Bag!")
  end
end

d = Manufactory.new.draw

#=>copyright: Ninja Company!
#=>I love my Bag!
```

Diff between those two examples, it can be easily find out that, there's only
one defferent bewteen.

* **Example1**: Method has been passed and called it later
* **Example2**: Whole object instance has been passed and call the method later.

And from the output, the simple idea could be generated:

* **Example1**: When our code saved a direct reference to a method and then called it later, 
it missed out on any changes to that method which occurred between when the reference 
was saved and when the method was executed.

* **Example2**: When we changed the code to hand over the whole instance and call method later,
it got the most up-to-date implementation of that responsibility.By doing that, 
objects are assured of getting the current, correct behavior no matter what

**In conclusion**, if you’ve ever wondered why we don’t pass references to methods 
around more often in Ruby, now you know.

## What I learned:

#### **Compromise**

Both Functional and Object-Oriented has it's own preference, but
sometimes, it's hard for us to get them all as those ideas can be divergent, 

And I believe this is an common case in our daily life as well.

**Be brave enough to compromise youself to focus on one idea and give up the others. that
does not means lost, that's balance.**

## Reference:
* [Presentation from RubyConf-China](https://blog.oyanglul.us/slides/functional-ruby.html) -- inspired by this presentation.
* [Method and
  Message](https://www.rubytapas.com/2012/10/17/episode-011-method-and-message/)
  -- basic idea is from there, follow up this screencast if you want to know more.

