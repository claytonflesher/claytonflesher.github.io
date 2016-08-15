---
layout: post
title: Learning Inject by using it to rewrite Enumerables, Part 2
---

# Learning Inject by using it to rewrite Enumerables, Part 2

Last time we began the process of rewriting Ruby's Enumerable methods using inject/reduce in both Ruby and JavaScript.

We left off with..

## each_with_object

This method is very similar to the core functionality of inject.

Ruby documentation: Iterates the given block for each element with an arbitrary object given, and returns the initially given object.

Let's say we want to build an array by doubling every value in another array.

With each_with_object:

```ruby
evens = (1..10).each_with_object([]) { |i, a| a << i*2 }
```

You can literally get the exact same behavior from inject by switching the order of `i` and `a`.

```ruby
evens = (1..10).inject([]) { |a, i| a << i*2 }
```

With JavaScript:

```javascript
evens = [1,2,3,4,5,6,7,8,9,10].reduce(function(memo, n) {
  return memo.concat(n*2);
}, []);
```


## entries

Just returns an array containing the items in the array.


## find_all/select

From documentation: Returns an array containing all elements of enum for which the given block returns a true value.

```ruby
(1..10).find_all { |i| i % 3 == 0 } #=> [3, 6, 9]
```

With inject:

```ruby
(1..10).inject([]) { |memo, i| i % 3 == 0 ? memo << i : memo }
```

With JavaScript:

```javascript
[1,2,3,4,5,6,7,8,9,10].reduce(function(memo, i) {
  if (i % 3 == 0) {
    return memo.concat(i);
  } else {
    return memo;
  }
}, []);
```


## find_index

From Documentation: Compares each entry in enum with value or passes to block. Returns the index for the first for which the evaluated value is non-false. If no object matches, returns nil.

```ruby
(1..10).find_index  { |i| i % 5 == 0 and i % 7 == 0 }  #=> nil
(1..100).find_index { |i| i % 5 == 0 and i % 7 == 0 }  #=> 34
```

The only way I could figure out how to do this is to cheat and use `break`. That violates the spirit of what I'm trying to do. If you can think of a way, let me know.


## grep

From Documentation: Returns an array of every element in enum for which Pattern === element. If the optional block is supplied, each matching element is passed to it, and the block’s result is stored in the output array.

So, I'm going to be honest. I didn't realize this method even existed until I read it for the purposes of this blog post. That said, it looks super useful. It's funcitons basically as an `Enumerable#map` that just takes a regular expression as an additional argument instead of having to us it inside of the block.

```ruby
c = IO.constants
res = c.grep(/SEEK/) { |v| IO.const_get(v) }
res                    #=> [0, 1, 2]
```

So, you either have to hard code the regular expression into the block, or use an additional map function.

```ruby
c = IO.constants
res = c.inject([]) do |memo, v|
  if v =~ /SEEK/
    memo << IO.const_get(v)
  else
    memo
  end
end
res                    #=> [0, 1, 2]
```

```javascript
["hello", "hello_world", "goodbye"].reduce(function(memo, v) {
  if (/hello/.test(v)) {
    return memo.concat(v);
  } else {
    return memo;
  }
}, []);  // ["hello", "hello_world"]
```


## grep_v

From Documentation: Inverted version of #grep. Returns an array of every element in enum for which not Pattern === element.

switch the order of the if/else in the example above.


## group_by

From Documentation: Groups the collection by result of the block. Returns a hash where the keys are the evaluated result from the block and the values are arrays of elements in the collection that correspond to the key.

```ruby
(1..6).group_by { |i| i%3 }   #=> {0=>[3, 6], 1=>[1, 4], 2=>[2, 5]}
```

I was expecting this to be incredibly messy, but it turned out to be just meh...

```ruby
(1..6).inject({}) do |memo, n|
  memo[n % 3] ? memo[n % 3] << n : memo[n % 3] = [n]
  memo
end
```

It gets pretty rough in JS.

```javascript
[1,2,3,4,5,6].reduce(function(memo, n) {
  if (memo[n % 3]) {
    memo[n % 3] = memo[n % 3].concat(n);
  } else {
    memo[n % 3] = [n];
  }
  return memo;
}, {});
```


## include?/member?

From Documentation: Returns true if any member of enum equals obj. Equality is tested using ==.

```ruby
(1..10).include?(4)   #=> true
(1..10).include?(500) #=> false
```

```ruby
(1..10).inject(false) { |memo, n| memo || n == 4 }
(1..10).inject(false) { |memo, n| memo || n == 500 }
```

```javascript
[1,2,3,4,5,6,7,8,9,10].reduce(function(memo, n) {
  return (memo || n == 4);
}, false);

[1,2,3,4,5,6,7,8,9,10].reduce(function(memo, n) {
  return (memo || n == 500);
}, false);
```


## lazy

Lazy is a cool thing. You should learn about and take advantage of it. It is outside of the scope of this blog post.


## max/max_by

From Documentation: Returns the object in enum with the maximum value. The first form assumes all objects implement Comparable; the second uses the block to return a <=> b.

```ruby
a = %w(albatross dog horse)
a.max                                   #=> "horse"
a.max { |a, b| a.length <=> b.length }  #=> "albatross"
```

The first one is returning "horse", because "horse" is highest up in the alphabet. The second is returning "albatross", because it's the longest.

This one returns a value without being passed a block, so let's skip that behavior and focus on.

```ruby
a = %w(albatross dog horse)
a.inject do |memo, n|
  (memo.length <=> n.length) != -1 ? memo : n
end
```

```javascript
["albatross", "dog", "horse"].reduce(function(memo, n) {
  if (n.length >= memo.length){
    return n
  } else {
    return memo
  }
})
```

## min/min_by

```ruby
a = %w(albatross dog horse)
a.inject do |memo, n|
  (memo.length <=> n.length) != 1 ? memo : n
end
```

```javascript
["albatross", "dog", "horse"].reduce(function(memo, n) {
  if (n.length <= memo.length){
    return n
  } else {
    return memo
  }
})
```
