---
layout: post
title:  "Design Pattern on Meta-Programming"
sub-title: "(Metaprogamming on Design Pattern)"
date:   2016-12-04 10:20:00 +1200
categories: English
tags: ruby programming English design_pattern

---

### Question:

When we think of Proxy in Design Pattern, How Proxy usually works?

Here's an basic example:

* Client Class implement Proxy class
* Proxy class usually has a way to call the target method from the Real class
* Method from Proxy class will add something before call the method of the Real class
* Most of the time, Proxy class and Real class has the same name for user to implements

  ```ruby
  caller = RealClassProxy.new(RealClass.new).target_method

  class RealClassProxy
    def initialize(object)
      @object = object
    end

    def target_method
      puts "Do something else"
      object.target_method
    end
    #...
  end

  class RealClass
    def target_method
      put "Doing the real thing!"
    end
  end
  ```


### Question Again:

* Is that prefect?
* Is their anything can be done better?

  Let's think of the follow scenario:

  What if the `Proxy Class` written after `Client Class`'s implementation? After implements
  `Proxy Class`, each of the `Client Class` have to change their code to call `Proxy Class`
  instead of the `Real Class`

### So, is there any ways to makes it better?

  The answer is: `using unbound method`

```ruby

module ProxyModule
  def proxy(method_name)
    original_method = instance_method(method_name)  # => #<UnboundMethod: Dictionary#anagrams_for>

    define_method(method_name) do |*args, &block|
       puts "do something in proxy with unbound method"
       original_method.bind(self).call(*args, &block)
    end
  end
end
class RealClass
  extend ProxyModule

  def initialize
  end
  def target_method
    puts "real thing!"
  end

  proxy :target_method
end

RealClass.new.target_method

#=> do something in proxy with unbound method
#=> real thing!

```

  Done, you implement the proxy, but nothing have to change after!

### What I learned:

* There's always a ways to Decoupling
* Meta methodology helps Decoupling
* What to makes your life easier, try using Meta methodology to every aspect of your life, (not just code)! Development meta recognition of yourself would makes your life much easier as this code.
* Design Pattern could be more powerful with Meta methodology
