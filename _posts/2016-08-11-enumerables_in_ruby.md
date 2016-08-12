---
layout: post
title: Learning Inject by using it to rewrite Enumerables
---

# Learning Inject by using it to rewrite Enumerables

I love Ruby's [Enumerable](http://ruby-doc.org/core-2.3.0/Enumerable.html). It's one of the most useful tools in the language. I've been messing around in JavaScript lately, and `Enumerable` was one of the things I was missing most.

I mentioned this to a programmer friend, and he told me that you can recreate all of Ruby's `Enumerable` methods using `Enumerable#inject` (also aliased as `Enumerable#reduce`), and that JavaScript has a reduce method.

That got me thinking. How difficult would it be to:

1. Recreate the rest of Ruby's enumerables using only `Enumerable#inject`?
2. Translate it to JavaScript?

Let's try it!

I'm going to do this for `Array`, because it's easiest for new programmers to follow in their head, and every langauge I know of has some concept of an array or linked list, but this works just as well for Ruby's `Hash` object and JavaScript's [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) with minor tweaks.

I'm also going to, at least in this post, skip the enumerables that, when called even with a block, return an `Enumerable`.

First, a little lesson on how `Enumerable#inject` actually works. It is a method that you can call on any object in which `ThatObject#each` is defined and the `Enumerable` module is mixed in.

When `ThatObject#inject` is called, it takes on optional `memo` as an argument and passes the memo and whatever the current value on the list is to the block, performing the logic and returning the `memo` as an accumulator.

So, for example, let's say we wanted to sum the numbers in `[1, 2, 3]`.

We could call `[1, 2, 3].inject(0) { |memo, number| memo + number }`.
There is a shorter way to do this, but I'm writing it this way because it's the most explicit and explains exactly what is going on.

In this case, we passed `0` as an argument, so that made `0` our initial memo. Initially, reduce passes `0`, our memo,  and `1`, the first item in our array, to the block. It adds them, returning the result as the NEW memo. Then it took that memo, passed it to the block with `2`, summed those, returned another new memo, etc. until it reaches the end of the array. The final returned memo is the result of our summed array.

You can do this without passing an initial `0` memo and it will still work, because by default `inject` uses the first item in the array as the memo and passes it to the second item in the array.

Ruby, because it is really nice, can take this magic a step further and you can just pass a symbol, such as `:+` as an argument instead of the memo, and it'll treat that symbol as the logic to be used in the block, with the first item in the array as the default memo. That's a little bit of magic that hides the behavior for the purposes of what we're demonstrating today. All of the examples are going to use a block for the logic, so you can most easily make out what is actually happening under the hood.

Let's get started.

## all?

Ruby documentation: Passes each element of the collection to the given block. The method returns true if the block never returns false or nil. If the block is not given, Ruby adds an implicit block of { |obj| obj } which will cause all? to return true when none of the collection members are false or nil.

Example: Let's say I wanted to know if all of the items in an array are greater than 10.

The enumerable would look like this:

```ruby
[11, 12, 11, 400, 21].all? { |n| n > 10 } # returns true
```

```ruby
[11, 9, 11, 400, 21].all?  { |n| n > 10 } # returns false
```

Our new one looks like this:

```ruby
[11, 12, 11, 400, 21].inject(true) { |memo, n| memo && n > 10 } # returns true
```

```ruby
[11, 9, 11, 400, 21].inject(true)  { |memo, n| memo && n > 10 } # returns false
```

In JavaScript, there is an `Array.prototype.every()` method that serves the same function, but we're going to do this with `inject`, because we can.
In JavaScript:

```javascript
[11, 12, 11, 400, 21].reduce(function(memo, n) { return memo && n > 10; }); // returns true
```

```javascript
[11, 9, 11, 400, 21].reduce(function(memo, n)  { return memo && n > 10; }); // returns false
```

Notice that we are passing the memo of `true` as the second argument to `Array.prototype.reduce()`. In JavaScript, you pass the callback function first, and the initial value of the memo, if needed, second.

Note quite as clean as in Ruby, but still pretty useful. You could even extract that function definition out and assign it to a descriptively named variable, to improve the readability over even that of Ruby's `all?`.

## any?

Ruby documentation: Passes each element of the collection to the given block. The method returns true if the block ever returns a value other than false or nil. If the block is not given, Ruby adds an implicit block of { |obj| obj } that will cause any? to return true if at least one of the collection members is not false or nil.

Example: Let's say I wanted to know if any of the items in an array are greater than 10.

The enumerable would look like this:

```ruby
[9, 1, 5, 13, 9].any? { |n| n > 10 } # returns true
```

```ruby
[9, 1, 5, 3, 9].any?  { |n| n > 10 } # returns false
```

Our new one looks like this:

```ruby
[9, 1, 5, 13, 9].inject(false) { |memo, n| memo || n > 10 }` returns `true`
```

```ruby
[9, 1, 5, 3, 9].inject(false) { |memo, n| memo || n > 10 }` returns `false`
```

In JavaScript, there is an `Array.prototype.some()` method that serves the same function, but we're going to do this with `inject`, because we can and because `some()` is a stupid name for it.
In JavaScript:

```javascript
[9, 1, 5, 13, 9].reduce(function(memo, n) { return memo || n > 10; }, false); returns `true`.
```

```javascript
[9, 1, 5, 3, 9].reduce(function(memo, n) { return memo || n > 10 ;}, false);` returns `false`.
```

## collect/map

Like inject/reduce, `collect` and `map` are aliases for one another in Ruby. Generally, you see `map` used, but some people prefer collect generally or in certain situations where it might be more descriptive. I prefer `map`.

Ruby documentation: Returns a new array with the results of running block once for every element in enum.
If no block is given, an enumerator is returned instead.

Example: Let's say I wanted to multiply every number in an array by 2.

The enumerable would look like this:

```ruby
[1, 2, 3, 4, 5].map { |n| n * 2 }
```

Our new one would look like this:

```ruby
[1, 2, 3, 4, 5].inject([]) { |memo, n| memo << n * 2 }
```

JavaScript has an `Array.prototype.map()`, but we're doing things the hard way, so let's do it with `reduce`.
In JavaScript:

```javascript
[1, 2, 3, 4, 5].reduce(function(memo, n) { return memo.concat(n * 2); }, []);
```

## collect_concat/flat_map

Another alias. I always use `flat_map`. Just out of curiousity, I searched Github for instances of `flat_map` and `collect_concat`. The first is more popular by over an order of magnitude. Use `flat_map` if you don't want to confuse people. Even Ruby's documentation examples prefer it.

Ruby documentation: Returns a new array with the concatenated results of running block once for every element in enum.
If no block is given, an enumerator is returned instead.

Example: Let's say I wanted to append a `1` at the end of ever sub-array of `[1, 2, 3], [4, 5, 6]]` and return a flattened array like this `[1, 2, 3, 1, 4, 5, 6, 1]`.

The enumerable would look like this:

```ruby
[[1, 2, 3], [4, 5, 6]].flat_map { |array| array << 1 } # returns [1, 2, 3, 1, 4, 5, 6, 1]
```

Our new one:

```ruby
[[1, 2, 3], [4, 5, 6]].inject([]) { |memo, n| memo += n << 1 }` # returns [1, 2, 3, 1, 4, 5, 6, 1]`
```

In JavaScript:

```javascript
[[1, 2, 3], [4, 5, 6]].reduce(function(memo, n) { return memo.concat(n.concat(1)); }, []);
```
It isn't pretty, but it works.

## count

Ruby documentation: Returns the number of items in enum through enumeration. If an argument is given, the number of items in enum that are equal to item are counted. If a block is given, it counts the number of elements yielding a true value.

Example: As usual, we're ignoring the non-block examples and using the ones that take a block.

Let's say we want to count the number of items in an array that are multiples of 3.

With count:

```ruby
[1, 2, 3, 4, 5, 6].count { |n| n % 3 == 0 } # returns 2
```

With our custom inject:

```ruby
[1, 2, 3, 4, 5, 6].inject(0) do |memo, n|
  n % 3 == 0 ? memo + 1 : memo
end # returns 2
```

In JavaScript:

```javascript
[1, 2, 3, 4, 5, 6].reduce(
  function(memo, n) {
    return n % 3 == 0 ? memo + 1 : memo;
  }, 0);
```

## cycle

This one is weird. It just runs forever if you don't pass it a limit. It's also kind of hard to figure out how to write in inject.

Ruby documentation: Calls block for each element of enum repeatedly n times or forever if none or nil is given. If a non-positive number is given or the collection is empty, does nothing. Returns nil if the loop has finished without getting interrupted.
`#cycle` saves elements in an internal array so changes to enum after the first pass have no effect.
If no block is given, an enumerator is returned instead.

Let's assume you're going to be passing it a number of times to run, and that'll be our memo when we use `inject`.

With cycle:

```ruby
[1, 2, 3].cycle(3) { |n| puts n } # prints 1 2 3 1 2 3 1 2 3
```

With our custom inject?

This seems, best I can tell, to be the exception to the rule. If you figure out a way to write this using inject/reduce, please let me know.

## detect/find

Ruby documentation: Passes each entry in enum to block. Returns the first for which block is not false. If no object matches, calls ifnone and returns its result when it is specified, or returns nil otherwise.
If no block is given, an enumerator is returned instead.

Let's say we want to find the first number in an array that is divisble by 5.

With find:

```ruby
[1, 2, 3, 4, 5, 6].find { |n| n % 5 == 0 } # returns 5
```

There is almost certainly a cleaner way to do this, but I got it working with...
Custom inject:

```ruby
[1, 2, 3, 4, 5, 6].inject(nil) do |memo, n|
  if memo
    memo
  elsif n % 5 == 0
    n
  else
    nil
  end
end
```

JavaScript has an `Array.prototype.find()`, so please don't do this in the wild, but...

In JavaScript:

```javascript
[1, 2, 3, 4, 5, 6].reduce(function(memo, n) {
  if (memo) {
    return memo;
  } else if (n % 5 == 0) {
    return n;
  } else {
    return null;
  }
}, null);
```

## drop

Drop is useful in Ruby, but it doesn't take a block. Let's skip it and move on to the much more useful...

## drop_while

Ruby documentation: Drops elements up to, but not including, the first element for which the block returns nil or false and returns an array containing the remaining elements.
If no block is given, an enumerator is returned instead.

Let's say we want to remove every item from our array leading up to 3.

With drop_while:

```ruby
[1, 2, 3, 4, 5, 1].drop_while { |n| n < 3 } # returns [3, 4, 5, 1]
```

With inject:

```ruby
[1, 2, 3, 4, 5, 1].inject(nil) do |memo, n|
  next if memo.nil? && n < 3
  memo ? memo << n : [n]
end
```

With JavaScript:

```javascript
[1, 2, 3, 4, 5, 1].reduce(function(memo, n){
  if (memo == null && n < 3){
    return null;
  } else {
    return memo ? memo.concat(n) : [n];
  }
}, null);
```

## each_cons

Ruby documentation: Iterates the given block for each array of consecutive <n> elements. If no block is given, returns an enumerator.

You can do this with inject, but it requires passing more than just a single value as the memo.

## each_entry

Ruby documentation: Call block once for each element in self, passing the element as a parameter, converting multiple values from yield to an array.

I might try to implement this one, later. It's...complicated, and not informative at the scope of this blog post without seriously getting into the weeds on things like `yield`.

## each_slice

See each_cons

## each_with_index

Calls block with two arguments, the item and its index, for each item in enum. Given arguments are passed through to each().

With each_with_index:

```ruby
hash = Hash.new
%w(cat dog wombat).each_with_index { |item, index| hash[item] = index }
hash #=> {"cat"=>0, "dog"=>1, "wombat"=>2}
```

Ruby's each_with_index is actually the perfect example case where reduce/inject should almost universally be preferred. You can get exactly the same behaviour, without needing the hash declaration in the line above. But anyway, here's an example.

With inject:

```ruby
hash = Hash.new
%w(cat dog wombat).inject(0) do |memo, animal|
  hash[animal] = memo
  memo += 1
end
```

With JavaScript:

```javascript
hash = {};
["cat", "dog", "wombat"].reduce(function(memo, n){
  hash[n] = memo;
  return memo += 1;
}, 0);  
```

I'm going to cut it off, there for now. This blog post is getting unweildy and I've already been sitting on it for months. This should give you some insight into how reduce/inject, and some of Ruby's iterators work. I'll try to come back to this post some time in the future and make it into a series.
