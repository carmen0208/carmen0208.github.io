---
layout: post
title:  "Hack and Tricks"
date:   2016-12-02 17:25:03 +1200
categories: English
tags: ruby programming English

---

## Question? Who is clever? you or machine? or you?

Most developers(including me) become confident when we become an more skilled developer.
We playing some tricks to avoid the code quantity which makes us proud of it.

We plays a lot util we stuck by some advance code we made and here's some example:

Sometimes, when we set hash, we could like to add a default value to avoid makes some trouble of our further implementation. like this:
 [comment1: example](#supplementary	comments)

 But sometimes, it could be frustrated

```ruby
# We makeing a default array into hash in order to get those expected values:
# people["carmen"] = ["carmen1", "carmen2", "carmen3"]
# people["somebody"] = "somebody1", "somebody2"]
# people["nobody"] = []

default = []
people = Hash.new(default)
people["carmen"] << "carmen1"
people["carmen"] << "carmen2"
people["carmen"] << "carmen3"
people["somebody"] << "somebody1"
people["somebody"] << "somebody2"

#Problems:
p people["carmen"] #=> ["carmen1", "carmen2", "carmen3", "somebody1", "somebody2"]
p people["somebody"] #=> ["carmen1", "carmen2", "carmen3", "somebody1", "somebody2"]
p people["nobody"] #=> ["carmen1", "carmen2", "carmen3", "somebody1", "somebody2"]
```

It is tricky as even no body has a value, and that's not we expected. Let's dig into it and we find out the reason:

```ruby
p default #=>  ["carmen1", "carmen2", "carmen3", "somebody1", "somebody2"]
p people #=>  {}
```

it's seems like a Brain sharp but you fingered that, the things that you really manipulate, is the default list instead of hash.

## So？ how to fix it?

There's 2 ways but I recommend the last one.

* Option 1:

  ```ruby
   default = []
   people = Hash.new(default)
   people["carmen"] += ["carmen1"]
   people["carmen"] += ["carmen2"]
   people["carmen"] += ["carmen3"]
   people["somebody"] += ["somebody1"]
   people["somebody"] += ["somebody2"]

    #Problems:
   p people["carmen"] #=> ["carmen1", "carmen2", "carmen3"]
   p people["somebody"] #=> ["somebody1", "somebody2"]
   p people["nobody"] # => []
   p people #=> {"carmen"=>["carmen1", "carmen2", "carmen3"], "somebody"=>["somebody1", "somebody2"]}
   p default #=> []
  ```

* Option 2 - Put default value into block

  ```ruby
   people = Hash.new do |hash, default_value|
     hash[default_value] = []
   end

   people["carmen"] << "carmen1"
   people["carmen"] << "carmen2"
   people["carmen"] << "carmen3"
   people["somebody"] << "somebody1"
   people["somebody"] << "somebody2"

   p people["carmen"] #=> ["carmen1", "carmen2", "carmen3"]
   p people["somebody"] #=> ["somebody1", "somebody2"]
   p people["nobody"] # => []
   p people #=> {"carmen"=>["carmen1", "carmen2", "carmen3"], "somebody"=>["somebody1", "somebody2"]}
  ```


### What I learn?

* Once you feel you confident about something, Be care, because something danger would happen.
* Human is not the most powerful species in the world.
* Keep learning


******************************

### supplementary	comments

* The reason why default value in Hash is useful

  ```ruby
  text = <<END
  Most developers become confident when we become an more skilled developer. which
  We playing some tricks to avoid the code quantity which confident proud us proud of it.
  We playing confident tricks to avoid the code quantity which makes us skilled of it.
  proud proud
  END

  counts = Hash.new(0)

  text.split.map(&:downcase).each do |word|
    counts[word] += 1
  end

  counts
  #=>{"most"=>1, "developers"=>1, "become"=>2, "confident"=>3, "when"=>1, "we"=>3, "an"=>1, "more"=>1, "skilled"=>2, "developer."=>1, "which"=>3, "playing"=>2, "some"=>1, "tricks"=>2, "to"=>2, "avoid"=>2, "the"=>2, "code"=>2, "quantity"=>2, "proud"=>4, "us"=>2, "of"=>2, "it."=>2, "makes"=>1}

  ```
