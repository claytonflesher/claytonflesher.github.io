---
layout: post
title: Using Sinatra
---

#Using Sinatra

Last week, I got a basic To Do app working in Sinatra. 

For those who haven't used it before, [Sinatra](http://www.sinatrarb.com/) is a [DSL](http://en.wikipedia.org/wiki/Domain-specific_language) built on top of Ruby that helps you build web applications.

Now, at this point you may be asking me, 'Clayton, isn't that what Rails does?' Well, yes it is. But Rails is a behemoth with a lot of magic that happens under the hood, magic that very few people really understand.

Sinatra doesn't build directories for you. You write all of your own routes. In short, you learn more about how web applications are built using Sinatra than you do Rails.

Basically, Sinatra is a web app builder that doesn't hold your hand, let's you see most of what is happening, and doesn't punish for going off the Rails, so to speak.

##Introducing Routes

Before you get started with Sinatra, there is a little bit you're going to need to really understand. The most important is how [HTTP routes](http://www.sinatrarb.com/intro.html) work.

There are a number of route commands, but for our purposes today, we are only going to use two: `get` and `post`.

What you need to know about them, is that when you use `get`, you're telling Sinatra to display the page. When you use `post`, you're usually telling Sinatra to take some data provided by the user and do something with it.

So, for example. Let's say I have a simple Login page. Remember, we're using Sinatra, so we are writing our own authentication. Devise isn't a thing, here.[^1] 

So we need to make some HTTP routes for a Login page. They might look something like this...

```ruby

get "/login" do
  title = "To Do / Login"
  erb :login, locals: {title: title}
end

post "/login" do
  title = "To Do / Login"
  user = db.authenticate(email: params[:email], password: params[:password])
  if user
    session[:email] = user.email
    redirect "/"
  else
    erb :login, locals: {title: title}
  end
end

```

For the new to Sinatra, there is a lot to take in there, but let's just focus on the beginning of the two blocks first.

The <code>get "/login" do</code> line is just telling us that we are going to display something.

In this case, the something is an erb file (more on that later) that takes one argument, the title, that will be displayed in the tab/header bar of the HTML file.

Our <code>post "/login" do</code> line is going to do quite a bit more, and this is usually going to be the case.

The <code> post "/login" do</code> line is telling us that everything below is going to be doing stuff like taking data from forms and using it, and then either rendering the page or redirecting to another page.

##Displaying the page

Now that we have our HTTP routes set up for our login page, we have to write a few more things. First, our `get` needs a page to land on. That's where `erb` comes in.

ERB, which stands for Embedded ruby, allows you to write Ruby in your views files, and they'll render it as HTML.

We need to create a `views` directory. Right now, we need to put two files inside it: a `layout.erb` and a `login.erb`.

Our layout file will look like this.

```html
<!DOCTYPE html>
<html>
  <head>

    <meta charset="utf-8">
    <title><%= h title %></title>
    <link rel="stylesheet" href="css/todo.css" type="text/css" media="all" charset="utf-8">
  </head>
  <body>
    <%= yield %>
  </body>
</html>
```

This layout page allows us to render the whole website using the same css stylesheet without having to add it to every other ERB file. Our `login.erb` will then use the same css formatting as our `dashboard.erb` and our `sign_up.erb`.

*(Important note: On the `title` line you'll see that the input is not just `title` but `h title`. The `h` is to escape the code that is being displayed. Any time you have data that could feasibly have come from a user passing between the HTML layer and the ruby code, be sure to escape it. This will help prevent users from inserting code into your forms and gaining control of your entire application.

To do this, you have to enable the ERB::Util helper in your routes file. It'll look something like the following.)*

```ruby

helpers do
  include ERB::Util
end

```

Now that we have a layout, we can create our Login page.

In it's simplest version, it'll need to have a form that calls our `post "/login" do` and sends it an email and a password.

It might look like this.

```html
<h1>Login Page</h1>
<form id="login" action="/login" method="POST">
  <input type="text" placeholder="email" name="email"><br>
  <input type="password" placeholder="password" name="password"><br>
  <input type="submit" value="login">
</form>
```

That should allow us to fill out a form, and then submit it to see if we are a valid user. Of course, we'll have to write a Sign Up page as well, so that we can create that user. But right now let's assume we've got that and we are just trying to authenticate our user.

##Sending to the Database

Now that we have a form and it is trying to send us information, we need to take that information and verify that we have a valid user.

That's going to require two ruby objects, a User and a Database. 

For our database, we are going to be using a basic system from the Ruby Standard Libarary called [PStore](http://ruby-doc.org/stdlib-2.2.2/libdoc/pstore/rdoc/PStore.html). If you're going to be deploying an application with a lot of database robustness and versatility, PStore is probably not a great idea, but for our purposes on this app, it's just fine.

Pstore has a very simple interface. You just call `.transaction do` on it with a `true` or `false` argument that determines if it is read only or not. If you don't give it an argument, the default is false, or read-write. Everything else is normal ruby code. So, our `Database.authenticate` method is going to look something like this.

```ruby

def authenticate(email:, password:)
  db.authenticate(true) do
    if db[:users].include?(email) && db[:users][email].password == password
      db[:users][email]
    else
      nil
    end
  end
end

```
It's that simple to write a basic user authentication system. There are probably some edge cases, and you'll have to create a check in your sign up to make sure that existing users aren't deleted when someone tries to sign up with an email already in the database, but otherwise it's done.

Assuming we've built a User object, and that User is saved to our database, the authenticator will now be able to check if the email and password provided are valid.

##Password Encryption

One of the things we are going to want to do is encrypt our users' passwords. 

Fortunately, Ruby gives us a dead simple way to do this as well.

If we start with a very basic User object, that contains maybe three things, an email, a password and the list of ToDo items, we can do it with two lines, a `require 'bcrypt'` and a simple call to the BCrypt object.

[BCrypt](https://github.com/codahale/bcrypt-ruby) is a ruby gem that offers robust password encryption.

Initially, our User object is going to look something like this...

```ruby
require 'bcrypt'

module Todo
  class User
    def initialize(email: nil, password: nil)
      @email    = email
      @password = BCrypt::Password.create(password)
      @list = [ ]
     end

    attr_reader :email, :password, :list
  end
end

```

The `@password` instance variable is smarter than you might think. By making it into a BCrypt object, I can use a simple `==` evaluator to check if it is valid.

So, for example:

```ruby

my_password = BCrypt::Password.create("Calvyn")
my_password == "Calvyn"           #==> true
my_password == "Anything else"    #==> false

```
Apparently, BCrypt actually uses Ruby's open classes to redefine the `==` method, allowing it to make such a simple evaluation without breaking the encryption. Don't ask me how, but it's pretty cool and super useful.

I'm going to cut the Sinatra tutorial off there. I encourage you to give it a look. My favorite thing about Sinatra, especially with small applications, is that it doesn't create a bunch of unnecessary directories and paths in your code. You only get out what you put in.

If you want to see my completed, working ToDo Sinatra app, I have it up on Github [here](https://github.com/Calvyn82/todo).

[^1]: 
Sinatra has something else you can use for user authentication called [Warden](https://github.com/jsmestad/sinatra_warden). We're building our own today. The whole point of me using Sinatra is so that I get closer to the code and have a better understanding of how the application is actually functioning. Writing my own user authentication is part of that, even if it doesn't provide the same detail of coverage as a gem might.
