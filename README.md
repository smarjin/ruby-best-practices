## Ruby. Best Practices.

### 1. Dynamic programming languages. Problems with big projects.
The purpose of this guide is not to teach how to program or design patterns per se, but rather serve as a reference [or a reminder] that most of the times when the design of an app, or a specific flow of that app gets clunky, it is a sign that something might not be quite right or that something could be improved. Obviously, even the most ideal app that reflects "a real business" can get more complex over time and some things are just the way they are. But developers must ensure that they do a good breakdown of the domain knowledge they possess and the acquired techniques.

While design patterns are [_mostly_] language agnostic concepts, the dynamic programming languages are the ones where things get more difficult to follow when the app grows. And that is a natural thing when there are no static types in place. This suggests that something else should be considered, or something that already exists has to be improved. Perhaps even consider it [the dynamic programming language] as a particular case when it comes to best practices? Because as the code grows, it gets harder to see what a certain entity is and how did it end up at a certain place. Is it something provided through the dependecy injection mechanism? Or is it an attribute? Or maybe a method that evaluates something and caches it? Do we need some meaningful _symbols_, prefixes or suffixes to better reflect what is what? When the class under investigation has around 200 lines it can get harder to find out. When there is a mesh of objects interactions with their blueprints defined in different classes [or, files per se] and the programmer has to navigate through all of them to make sense of what certain _variables_ are meant to be, it can get very annoying and tedious.

Very often, the SOLID principles are employed correctly, hence this concept is not disregarded. But the _single responsibility_ principle, for example, can easily become your enemy in a big code base when there is an _abuse_ of files [aka classes], or even method calls within the same class. Why? Again, because of attention span and knowing exactly "_what is what?_". This might not be a problem for the interpreter, but it is for us,  those who write and read. This can only happen when there is no:
1. systematic way for _denominalization of things_ (a workaround for the lack of static types);
2. tools to check the types (e.g: sorbet, rbs, etc.);
3. static typing mimicking tool or approach (e.g: method always returns the same type)

The so called "_abuse_" of separate entities that encapsulate single responsibilities pops up when there is a huge pool of these entities. Imagine having a large flow or a complex feature in a web app, that will inevitably lead to a lot of classes, hence instances. All those classes encapsulate certain behavior and responsibilities. If this feature is written in such a way that it only follows the SOLID principles then the prognosis of failure is not even required as failure becomes religion. There will be a huge web of objects sending messages to each other and navigating through that _naive infrastructure_ will be the programmer's quest. How productive is this?

In the end, this is a part of the software engineer's job, to analyze, write and _read_ code. The question is: _how can this process be improved?_

### 2. Design Patterns
One of the most powerful tools are Design Patterns. When used right...

It is beyond the scope of this guide to make a voyage through the entire galaxy of design patterns and talk about their mise-en-scenes. The intention is to highlight some patterns that can help to flatten out the design complexity and solve some of the issues that appear in an OOP [or imperative] programming language when used incorrectly or not carefully. Obviously the OOP paradigm has a lot of benefits but it has its own disadvantages, namely the unintentional change of a _state_. Or, the _intentional_ change of a state there where it should not be done. Hence all the debates about _functional_ vs _object oriented_ styles on all those "watering holes" forums. 

This section brings up some design patterns that are capable to _unmesh_ the chaotic world of objects and break it down. But nothing takes more time than _to think_ and then _write_. This is the main prerequisite required to simplify things in long term. Any process can be quantized in sub-processes. Each sub-process can be quantized subsequently. The art is to know when enough is enough. 

#### 2.1 Mediator

The `mediator` design pattern [or any hybrid of sorts] seems to be a good fit to _group_ those _scattered_ "one responsibility" objects. It offers the required mechanism to avoid having: 
- too deeply nested folders;
- huge interdependency (or a long chain of calls involving many instances);
- too many flows of control

Even if files are placed in the same directory, the chain of calls is not guaranteed to be a short one. There can be 10 files in the same directory, each defining a class. Then those classes get involved in a flow. There's a lot of chances to see those classes being [more or less] subsequently instantiated in each class. That builds an invisible, but fat Russian doll.

This could be avoided by using the `mediator` design pattern. It is a behavioral design pattern that lets you reduce chaotic dependencies between objects. The pattern restricts direct communications between the objects and forces them to collaborate only via a mediator object. 

It suggests that you should cease all direct communication between the components which you want to make independent of each other. Instead, these components must collaborate indirectly, by calling a special mediator object that redirects the calls to appropriate components. As a result, the components depend only on a single mediator class instead of being coupled to dozens of their colleagues.

Here's how it looks:
```ruby
component1 = Component1.new
component2 = Component2.new
componentN = ComponentN.new

Mediator.new(component1, component2, ..., componentN).call
```

For more information visit the following [**link**](https://refactoring.guru/design-patterns/mediator). There's a very good example with a `DialogBox` that behaves like a mediator for the components it consists of.

The `mediator` design pattern solves the _interdependency_ problem. There won't be a huge chain of subsequent calls and delegations to other objects anymore. This approach also suggests that components must already be in place when the process starts. That means less _states_ because those components will have a method as an entrypoint that might need some input [perhaps the data of another component even] but the result of each component [or subprocess] call can be stored in the _mediation scope_, hence repealing the need to have states in the deeper levels.

> Each component can be a [**PFAAO**](https://kellysutton.com/2017/09/13/embracing-functional-programming-in-ruby.html) entity [which responds to __call__ for example]. So as a result, the risk of state mutation is highly reduced.

#### 2.2 PFAAO (Pure Function As An Object)

Embrace the functional world. This principle is a _purely_ functional concept as it is an analogue of a mathematical function. But what is it about?

Taken from Wikipedia:
> **Pure function:**
In computer programming, a pure function is a function that has the following properties:
> 1) The function return values are identical for identical arguments (no variation with local static variables, non-local variables, mutable reference arguments or input streams).
> 2) The function application has no side effects (no mutation of local static variables, non-local variables, mutable reference arguments or input|output streams).

And that's all we care about really. The implementation is simple and easy to understand too. Here are the steps:
1. Create a class that implements a public method (e.g: _call_) and instantiates an object of the same class;
2. Make the constructor private

Example:
```ruby
class APfaaoExample
  # @param something [Object]
  #
  # @return [SameTypeAlways<value>]
  def self.call(something)
    new(something).call
  end

  # Receives the input and stores it (but it is safe to do so)
  # Consider it like a local variable for a function (which is stored in stack)
  # Freezes it to make sure no mutations are done to `something` (other methods of protection can be used too)
  #
  # @param something [Object]
  def initialize(something)
    @something = something.freeze
  end
  private_class_method :new

  # @return [SameTypeAlways<value>]
  def call
    task1.result + task2.result + task3.result
  end
  
  # This is also an example when it is required to cache a slow operation
  # So some tasks rely on same result for example (slow_operation)
  def task1(something); slow_operation(something); end
  def task2(something); SomeOtherPFAAO.call(something); end
  def task3(something); slow_operation(something); end
  
  # This is a heavy operation but it is fine to cache it because it depends on `something` which is always the same.
  def slow_operation(something)
    @slow_operation ||= SomeRemoteCallPFAAO.call(something)
  end
end
```

Note that it is still possible to mutate the state of `@slow_operation`. But using PFAAO and mutating objects is insane. One can freeze it though, if one doesn't trust oneself. This classes should be kept of rational sizes. It should respect the single responsibility principle as well.

By using this principle one achieves the referential transparency out of the box. Consider the following:

```ruby
res = APfaaoExample.call(1)
res.call # => NoMethodError (private method `call' called for #<APfaaoExample:0x00007fa381083230>)
```

As one can see `res` cannot be mutated because everything is `private` at the instance level.

> If this class implements the "to_proc" method, it will also allow the syntax sugar for using the unary operator when filtering Enumerators:

```ruby
# For the sake of creating an example:
APfaaoExample.class_eval do
  def self.to_proc
    method(:call).to_proc
  end
end
multiple_results = [1,2,3].map(&APfaaoExample) # be careful with how many objects are being created when calling :to_proc
```

For more details [**check this link**](https://kellysutton.com/2017/09/13/embracing-functional-programming-in-ruby.html).

#### 2.3 The Facade
Combine The Facade and The mediator design patterns (keep the design as flat as possible)
        1. https://en.wikipedia.org/wiki/Facade_pattern

#### 2.4 Value objects

There are code bases where `String` objects are flying like saucers. While this can be a breathtaking view in real life, it is pure madness in a code base. It can even get dramatic sometimes. In some places people can decide to use downcased representations of a `String` object, in other places the upcased ones. Who’s to blame? What if we need some behaviour attached to that entity too? Patching `String` is not an option. 

Using custom value objects also gives the possibility of defining custom logic of what that entity means and add more meaning to how it should be compared to other instances of the same kind. One can think of it as a simple decorator object to a plain `String` value. If one goes a bit further then Pattern Matching comes into the picture as well. Consider these `Value` objects as tiny `case classes` in `Scala`. The keyword here is **tiny**. One must be careful with HOW MUCH behaviour gets attached to the `Value` object, otherwise is it a `Value` anymore?

Some of the key advantages are that:
- they remove duplication;
- operations on particular data get gathered in a single place, instead of being disperse throughout the code

[**Check this out**](https://revs.runtime-revolution.com/value-objects-in-ruby-on-rails-9df64bc8db34) for more details about this design pattern.


### 3. NilClass is not your friend
Consider using the `NullObject` design pattern instead. The absence of something is not necessarily associated with `nil`. We can establish ourselves what absence means. Whence, it is natural that this approach may also solve some referential transparency problems scattered around in the codebase. It also suggests that all methods should always return the same type, thus have a more predictable behaviour.

Introducing `sorbet`, `rbs` or other similar tools will prove to be easier eventually if methods always return a certain Type. `NilClass` is also considered to be a low level type in other languages like Scala. In Ruby, Python, Java, etc. one constantly needs to be wary that values might be null. In Haskell, if for some value it's intended that it might not be defined, then one must always make this explicit by wrapping the type in a Maybe. By doing this, one makes sure that anybody trying to use the value must first check whether it's there. Not possible to forget this and run into a null-reference exception at runtime!

```ruby
class SomeClass
  # @param input [SomeType]
  # @raise [JustAnotherCustomError]
  #
  # @return [SomeCustomType, NilClass] -> how good is this?
  def self.call!(input)
    return if input.blank?
    some_other_resource[input.value] || raise JustAnotherCustomError
  end
end

# Compare this:
res = SomeClass.call!(OpenStruct.new(value: 1)) # => nil or Some(value)
do_something(res) if res.present?

# To this:
# Always returns a CustomType that responds to :call.
# Sometimes it can be a NullCustomType(CustomType) that responds to :call as well.
res = SomeClass.call!(OpenStruct.new(value: 1))
do_something(res.call)
```

The second option is always predictable. It also mimics the `static` programming style and allows more generic implementations. Another potential problem of `SomeClass.call!` is that if this call is made somewhere in depth it will bubble up the error to upper layers, so in the end, everyone knows about the specifics of this call.

### 4 Re-Raising errors is too old school
Raising, re-raising, rescuing… When especially done in different layers of abstraction… It bubbles up the problem from the lowest layer to the highest. Whence all levels on top of the lowest one must know about those errors and handle them accordingly. Doesn’t it smell? It asks to be replaced by a monad. One should always strive to have as little levels of code nesting as possible. Perhaps we should rethink our establishment of: 1 file -> 1 class. There are cases when there are private inner classes (helper classes) or “interfaces” that belong to the same place and/or main type aka class. Having more stuff in a single place will improve the speed of getting to know a new project. We should probably review the MethodLength value that gets assigned to rubocop.

### 5. dry-rb
    1. Use the relevant dry-rb gem to get the most out of the functional concepts in Ruby.
4. Project Structure
    1. Divide the project per domain and not per layer of abstraction (e.g: Domain vs Rails MVC) 
    2. https://currencycloud.slack.com/archives/C01FEMWGQ1H/p1631126849016000
5. Get more use of Pattern Matching
    1. Avoid having complex data flow controls. That’s a smell of a poor design. It is also harder to maintain. It has a higher cognitive load, especially on the next developer who will have to go through that area of code. 
6. Re-Raising errors is too old school
    1. Raising, re-raising, rescuing… When especially done in different layers of abstraction… It bubbles up the problem from the lowest layer to the highest. Whence all levels on top of the lowest one must know about those errors and handle them accordingly. Doesn’t it smell? It asks to be replaced by a monad. One should always strive to have as little levels of code nesting as possible. Perhaps we should rethink our establishment of: 1 file -> 1 class. There are cases when there are private inner classes (helper classes) or “interfaces” that belong to the same place and/or main type aka class. Having more stuff in a single place will improve the speed of getting to know a new project. We should probably review the MethodLength value that gets assigned to rubocop.
7. Performance
    1. When in doubt of performance use the `benchmark-ips` gem for comparisons
