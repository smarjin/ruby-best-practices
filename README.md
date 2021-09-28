## Ruby. Best Practices.

1. Design Patterns
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
