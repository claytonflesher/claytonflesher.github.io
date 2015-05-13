---
layout: post
title: Code Speed
---
#Code Speed

I recently checked out the Ruby Monday google hangout run by [Code Newbies](codenewbie.org). If you're new to development, or just to a language you are trying to learn, I suggest giving it a try.

In the hangout, the topic of optimizing code for speed came up, and I mentioned that there are some basic practices that Ruby developers can use to help produce faster code. I'm not an expert on this topic, but I can suggest a few basics I've learned from some of my old homework assignments.

##The long version

One area that can be particularly illustrative is when you are manipulating a large amount of data.

For example, let's assume we want to write a program that takes a string of characters, and returns every possible way to sort those characters into something that is recognized by a database of dictionary words.

Fortunately, we can do that by utilizing the dictionary on our computer.

If we don't care at all about how long it takes our code to run, it'll look something like this...

```Ruby 
class Reader
  def initialize(statement)
    @statement = statement.chars
  end

  attr_reader :statement

  def permutations
    statement.permutation.to_a.uniq.map do |array|
      array.join
    end
  end
end

class Scanner
  def initialize(statement, filename = "/usr/share/dict/words")
    @statement = statement
    @filename  = filename
  end

  attr_reader :statement, :filename

  def run
    list  = Reader.new(statement).permutations
    hits  = Array.new
    list.each do |word|
      File.foreach(filename) do |line|
        if line.strip == word
          hits << word
        end
      end
    end
  end
end

statment = ARGV.last
puts Scanner.new(statment).run
```
This program can get incredibly long to bogged down quite easily. The main reason is because of our <code>run</code> method.

What it is doing is taking every permutation of the word we feed it and checking it against the entire dictionary, one line at a time. That can be relatively quick for very short words, like 'rat'.

In my build, it takes the computer a not-great .231 seconds to figure out that 'rat', 'art' and 'tar' are all in the dictionary.

But when I run the code for a longer word, like 'desserts', it took my system 1 minute and 52.929 seconds to return 'desserts' and 'stressed' as valid words. This isn't going to make our users very happy. This is 2015. We can't have programs this simple taking that long to run.

##A little shorter

What if we refactored that <code>run</code> method a little bit? Instead of having it check the entire dictionary every time we wanted to see if a permuation was valid, what if we only checked the dictionary once, but checked each word in the dictionary against our array of permuations?

Checking a massive dictionary against a relatively small array is going to be a lot faster.

Our new <code>run</code> method might look something like this...

```Ruby
  def run
    list  = Reader.new(statement).permutations
    hits  = Array.new
    File.foreach(filename) do |word|
      if list.include?(word.strip)
        hits << word
      end
    end
    return hits
  end
```
When I swap out those methods and try 'rat' I get a runtime of .073 seconds. Much faster than 'rat' in our original version. But where the real advantage comes into play is with 'desserts'. The new version now takes 4.432 seconds. That's a serious improvement.

##Even faster

But what if we wanted to further improve our run-time? To get signficant improvements over what we already have, we are going to have to start making some serious changes to our code.

What if we created a new database out of the dictionary that was optimized for speed?

When we ran the program, we could check if such a database existed. If not, we could create it and store it in our repository. That wouldn't be very fast on that initial run, but every time we ran the program after that, we should get some serious speed improvements.

Here is our new version.

```Ruby
DATABASE = "database.txt"

class Builder
  def initialize
    @database = DATABASE
  end

  attr_reader :database

  def create_database
    File.open(database, "w") do |f|
      import.each do |key, value|
        f.puts "#{key}=>#{value.join("|")}"
      end
    end
  end

  def import(filename = "/usr/share/dict/words")
    dictionary = Hash.new
    File.foreach(filename) do |word|
      word.strip!
      normalized = word.chars.sort.join
      dictionary[normalized] ||= Array.new
      dictionary[normalized] << word
    end
    return dictionary
  end
end

class Searcher
  def initialize(user_input)
    @user_input = user_input.chars.sort.join
  end

  attr_reader :user_input

  def descramble
    if File.exist?(DATABASE)
      find_in_database
    else
      Builder.new.create_database
      find_in_database
    end
  end

  def find_in_database
    File.foreach(DATABASE) do |line|
      entry = line.split("=>")
      if @user_input == entry.first
        return entry.last.split("|")
      end
    end
  end
end

user_input = ARGV.last
check = Searcher.new(user_input)
puts check.descramble                     
```
So, what is this code doing exactly?

Like the others, it takes the user's input when the program is run, but this time when the user's input is passed to the <code>Searcher</code> object, the characters are sorted alpha-numerically and then joined back into a new string.

When the program prints to the command line, it calls <code>Seacher.descramble</code>. This method first checks to see if the database file exists.

Let's assume, for the sake of argument, that this database doesn't exist (It won't the first time you run this program). If not, then it runs the <code>Builder</code> and calls the <code>create_database</code> method on it. This method writes to '/database.txt' using a very special format.

The <code>create_database</code> method uses the <code>import</code> method to read every entry in the computer's dictionary file and creates a new <code>dictionary</code> variable of its own. This new dictionary is a hash. The keys of the hash are every entry of the dictionary sorted alpha-numerically, and the values are an array of the unsorted entries. So, every time the <code>import</code> method spots a new key that hasn't been created yet, it does so and adds the corresponding value. If the key has already been created, then it just appends the value to the existing value array. Once our hash has been constructed, it is printed to '/database.txt'.

Now that the database is built, what you have is a much shorter dictionary of <code>key=>array</code> pairs. 

The <code>Searcher.descramble</code> then proceeds from where it left off by calling <code>find_in_database</code>. It opens your new database and looks to see if there is a key that matches the <code>user_input</code>. If it finds one, then it prints to the command line the contents of the corresponding array.

##Checking final speed

If there is not a '/database.txt' file in our directory, and we run the code with 'rat', it takes my system .756 seconds to build the new database and check it for our matches. 

That's not very fast, but if we run it a second time with our new optimized directory, it takes about .050 seconds. That's pretty good.

When we try 'desserts' with the built directory, it takes almost exactly the same amount of time. With our new database, the size of the word we are checking for is no longer a serious limiting factor. The number of permutations are effectively irrelevant, because we are only searching for the sorted key value.

##Conclusions

So what have we learned? Here are a few key takeaways.

First, try to limit the number of times your program calls upon a database. Every time it has to load up a database and iterate through each line, we're wasting precious resources. 

Second, if possible, optimize the data for search. There are a number of ways to do this, and developers should be keeping an eye out for opportunities. If you have an app that has to build a search-optimized database when it boots, that's going to add some time to the initial load, but it might save a ton of runtime in actual use if the database is being called repeatedly.

This post is just touching the surface of what can be done. I focused on database interaction, because it is such an obvious example. If you want a much more detailed look at writing faster Ruby, I suggest checking out what the [Pragmatic Programmers](https://pragprog.com/book/adrpo/ruby-performance-optimization) have on offer. I haven't read this particular book...yet, but my experience is that the Prag Prog publishers have some of the best material available.

##Upcoming

There are a lot of ways to optimize speed, and Rubyists should always be on the lookout for relatively painless opportunites to improve. 

One example that came up in the google hangout Monday night was search-optimizing algorithms. Algorithms, such as binary search, when utilized in the appropriate situations, can improve the time it takes to search a large database by orders of magnitude. Some blog post in the near future, I'll focus on binary search in particular, focusing on what situations it can be used, and why it is so much faster, on average, than a linear search.

I'm also going to do a post on character encoding in Ruby. What is it, and why is it important?
