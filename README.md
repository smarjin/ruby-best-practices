## Ruby. Best Practices.

### 1. Style
The purpose of this guide is not to teach how to program or design patterns per se, but rather serve as a reference [or a reminder] that most of the times when the design of an app, or a specific flow of that app gets clunky, it is a sign that something might not be quite right or that something could be improved. Obviously, even the most ideal app that reflects "a real business" can get more complex over time and some things are just the way they are. But developers must ensure that they do a good breakdown of the domain knowledge they possess and the acquired techniques.

While design patterns are [_mostly_] language agnostic concepts, the dynamic programming languages are the ones where things get more difficult to follow when the app grows. And that is a natural thing when there are no static types in place. This suggests that something else should be considered, or something that already exists has to be improved. Perhaps even consider it [the dynamic programming language] as a particular case when it comes to best practices? Because as the code grows, it gets harder to see what a certain entity is and how did it end up at a certain place. Is it something provided through the dependecy injection mechanism? Or is it an attribute? Or maybe a method that evaluates something and caches it? Do we need some meaningful _symbols_, prefixes or suffixes to better reflect what is what? When the class under investigation has around 200 lines it can get harder to find out. When there is a mesh of objects interactions with their blueprints defined in different classes [or, files per se] and the programmer has to navigate through all of them, to make sense of what certain _variables_ are meant to be, it can get very annoying and tedious.

Very often, the SOLID principles are employed and correctly, hence this concept is not disregarded. But the _single responsibility_ principle, for example, can easily become your enemy in a big code base when there is an _abuse_ of files [aka classes], or even method calls within the same class. Why? Again, because of attention span and knowing exactly "_what is what?_". This might not be a problem for the interpreter, but it is for us,  those who write and read. This can only happen when there is no:
1. systematic way for _denominalization of things_ (a workaround for the lack of static types);
2. tools to check the types (e.g: sorbet, rbs, etc.);
3. static typing mimicking tool or approach (e.g: method always returns the same type)

The so called "_abuse_" of separate entities that encapsulate single responsibilities pops up when there is a huge pool of these entities. Imagine having a large flow or a complex feature in a web app, that will inevitably lead to a lot of classes, hence instances. All those classes encapsulate certain behavior and responsibilities. If this feature is written in such a way that it only follows the SOLID principles then the prognosis of failure is not even required as failure becomes religion. There will be a huge web of objects sending messages to each other and navigating through that _naive infrastructure_ will be the programmer's quest. How productive is this?

In the end, this is a part of the software engineer's job, to analyze, write and _read_ code. The question is: _how can this process be improved?_

### 2. Design Patterns
One of the most powerful tools are Design Patterns. When used right...

It is beyond the scope of this guide to make a voyage through the entire galaxy of design patterns and talk about their mise-en-scenes. 

###### The Facade and Mediator patterns
    1. Combine The Facade and The mediator design patterns (keep the design as flat as possible)
        1. https://en.wikipedia.org/wiki/Facade_pattern
        2. https://en.wikipedia.org/wiki/Mediator_pattern
    2. The PFAAO design pattern
        1. https://kellysutton.com/2017/09/13/embracing-functional-programming-in-ruby.html
    4. Value Object design pattern
        1. Strings are flying like saucers in our codebase. While this can be a breathtaking view in real life, it is pure madness in a codebase. Things can get hairy. In some places people can decide to use downcased representations of a String, in other places the upcased ones. Who’s to blame? What if we need some behaviour attached to that entity too? Patching String is crazy. https://revs.runtime-revolution.com/value-objects-in-ruby-on-rails-9df64bc8db34. But one must be careful with HOW MUCH behaviour gets attached to the Value Object, otherwise is it a Value anymore?
        2. https://medium.com/@dannysmith/little-thing-value-objects-in-ruby-c4745aeb9c07
        3. 
2. NilClass is not your friend
    1. Consider using the NullObject design pattern instead. The absence of something is not necessarily associated with `nil`. We can establish ourselves what absence means. Whence, it is natural that this approach may also solve some referential transparency problems scattered around in the codebase. It also suggests that all methods should always return the same type, thus have a more predictable behaviour. Introducing `sorbet`, `rbs` or other similar tools will prove to be easier eventually if methods always return a certain Type. `NilClass` is also considered to be a low level type in other languages like Scala. In Ruby, Python, Java, etc. one constantly needs to be wary that values might be null. In Haskell, if for some value it's intended that it might not be defined, then one must always make this explicit by wrapping the type in a Maybe. By doing this, one makes sure that anybody trying to use the value must first check whether it's there. Not possible to forget this and run into a null-reference exception at runtime!
3. Consider using dry-rb
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
