---
title: "Who ya gonna #call?"
layout: post
---

In Ruby, the `#call` method is most commonly used to call functions – lambdas, procs, blocks, and so on.

{% highlight ruby %}
# Convert a string into a URL-friendly version
parameterize = lambda do |s| 
  s.gsub(/\s+/, '-').gsub(/[^a-zA-Z\-]/, '').downcase
end

string = 'Hello, world!'
parameterize.call(string) # => "hello-world"
{% endhighlight %}

We can also call methods on classes[^functions-vs-methods]. Every object has a method named `#method` (stay with me here), which returns a `Method` object pointing to that particular method on that particular object. Just `Proc#call`, there is also a `Method#call`.

{% highlight ruby %}
class StringTitleizer
  def titleize(s)
    s.split(' ').map { |w| "#{w[0].upcase}#{w[1..-1].downcase}" }.join(' ')
  end
end

titleizer = StringTitleizer.new
titleize_method = titleizer.method(:titleize) # => #<Method StringTitleizer#titleize>
titleize_method.call(string) # => "Hello, World!"
{% endhighlight %}

This is great: `#call` can execute two fundamentally different blocks of code – a function and a method – using a common interface. This is what object-oriented programming is all about.

## We need to go deeper

There's nothing unique about `Proc` or `Method`. They're classes, just like everything else in Ruby. And we know how to define our own classes.

{% highlight ruby %}
# Repeats a string X times
class StringRepeater
  def initialize(count)
    @count = count
  end

  def call(string)
    string * @count
  end
end

# Create our repeaters
doubler = StringRepeater.new(2)
tripler = StringRepeater.new(3)

doubler.call(string) # => "Hello, world!Hello, world!"
tripler.call(string) # => "Hello, world!Hello, world!Hello, world!"
{% endhighlight %}

Now we have three completely different ways to muck with strings:

- A lambda function, `parameterize`
- An instance method, `StringTitleizer#titleize`
- An object, `StringRepeater`

What do they have in common? You guessed it, `#call`.

## Bringing it all together

Let's put our string manipulations to work by defining a generic `StringTransformer`.

{% highlight ruby %}
# Performs arbitrary string transformations
class StringTransformer
  def initialize(transform, string)
    @transform = transform
    @string = string
  end

  def to_s
    # Delegate the string manipulation to the transform
    transform.call(string)
  end
end
{% endhighlight %}

Simple enough. Let's use it.

{% highlight ruby %}
# Use the parameterize lambda function
parameterize_transformer = StringTransformer.new(parameterize, string)
parameterize_transformer.to_s # => "hello-world"

# Use the titleize instance method
titleize_transformer = StringTransformer.new(titleize_method, string)
titleize_transformer.to_s # => "Hello, World!"

# Use the StringRepeater class
double_transformer = StringTransformer.new(doubler, string)
double_transformer.to_s # => "Hello, world!Hello, world!"
{% endhighlight %}

The `StringTransformer` doesn't care what *type* of object `transform` is, as long as it responds to `#call` – [duck typing](https://en.wikipedia.org/wiki/Duck_typing) at work.

**This is the power of `#call`**. It's nothing more than a simple convention, but it's imposed by the Ruby standard library to provide a common interface for executing arbitrary blocks of code. 

Object-oriented patterns typically focus on abstractions that hide away the *type* of an object. In Ruby, it's more prudent to think about abstracting away the *behavior* of an object. You don't care about what an object *is*, you care about what it *does*.[^poodr]

Maybe you have a simple one-liner proc, or maybe you have a giant model entangled with complex business logic and dependencies. Maybe you have a function that literally does nothing. Anything is fair game when you hide its behavior behind a simple `#call`.

## Bonus: A spoonful of sugary syntax

This concept is so important that Ruby even has syntax dedicated to it, colloquially referred to as the *call syntax*, although it's not commonly used.

{% highlight ruby %}
# Equivalent to parameterize.call('herp derp')
parameterize.('herp derp') # => 'herp-derp'

# Equivalent to titleizer.call('once upon a time')
titleizer.('once upon a time') # => "Once Upon A Time"

# Equivalent to tripler.call('12345')
tripler.('12345') # => '123451234512345'
{% endhighlight %}

## Call me, maybe

Here are few ideas where it's handy to have substitutable code blocks.

- Implement the [Visitor pattern](https://en.wikipedia.org/wiki/Visitor_pattern) – e.g. walk a graph and perform an operation on each node
- Pass an asynchronous callback function as an argument – see [Faye extensions](http://faye.jcoglan.com/ruby/extensions.html) and note how `callback.call(message)` is required for every extension
- [Write a Rack app](http://rack.github.io/) – literally the only requirement is that it must respond to `#call`
- Implement a chain of sequential functions, where the output of one function is the input to the next – useful when dynamically adding/removing behaviors in the middle of the chain at runtime, or for implementing a "middleware" stack
- Provide sugary call syntax for common tasks – see [Trailblazer's Cells gem](http://trailblazer.to/gems/cells/api.html)

So, give me a `#call` sometime?

[^functions-vs-methods]: In this case, I'm using the term *function* to refer to an anonymous code block not attached to any one class, and *method* being a function belonging to a particular object or module.

[^poodr]: *[Practical Object-Oriented Design in Ruby](http://www.amazon.com/gp/product/0321721330)* by Sandi Metz really drives this point home – recommended reading for any Rubyist.