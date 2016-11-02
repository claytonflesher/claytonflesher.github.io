---
layout: post
title: Copy to System Clipboard in Elm
---

# How to Copy to the System Clipboard in Elm, and Why it Works

Since the release of Elm 0.17, there has not been an agreed upon way to copy to the system clipboard from inside of Elm. This article will show you how to do so, without having to make use of any Elm Native libraries, flash, or writing a bunch of JavaScript.

## Skip to a Code Example

If you want to cargo cult this, you can skip straight to a code example of this technique in practice, [here](https://github.com/JEG2/elm_clipboard_test)

## Copying to the System Clipboard


### HTML Setup

First, you will need to get [Clipboard.js](https://clipboardjs.com/), a JavaScript library, and include a call to it in your `<header>`. The easiest way to do this is by making use of one of the several [third-party CDN providers](https://github.com/zenorocha/clipboard.js/wiki/CDN-Providers).

Once you've sourced the Clipboard.js file, add the following to the body.

```html
<body>
  <script> ...Your Elm.whatever setup goes in here.</script>
  <script> var clipboard = new Clipboard('name-of-thing-to-trigger-the-copy')</script>
</body>
```
In this example, we're going to use `new Clipboard('.copy-button')`

That is all of the JavaScript setup you need. The rest you do in your Elm code.

### Elm Setup

Let's say you have a text field, and you want to copy the contents of it to the clipboard when you click a button.

Let's add an `id` to our input function, so our copy button knows what it is copying.

```
input [ type` "text", id "copy-me", onInput Change ] []
```

We'll also need a button to click, to trigger the copy to the clipboard, along with an attribute that Clipboard.js recognizes and knows to target. `attribute` in Elm takes two arguments, the name of the attribute, and value assigned to that name.

```
button [ class "copy-button", attribute "data-clipboard-target" "#copy-me" ] [text "Button Text"]
```

That should be it. Clipboard.js will create all of the listeners we need, and when we enter some text into our `input` and click on the `copy-button` button, it'll copy the text to our system clipboard.

## Why It Works

This is actually an interesting problem in Elm. One of the great things about Elm is that there aren't any [side-effects](https://en.wikipedia.org/wiki/Side_effect_%28computer_science%29). That gives you a ton of benefits, like code-readability, Elm's incredible error messages, and the knowledge that if your code compiles you're pretty much done.

Unfortunately, copying to the clipboard is, by definition, a side effect. That means you really can't do it from inside of Elm, without doing one of two things: Taking advantage of some JavaScript that isn't inside of Elm, or writing an Elm Native library that introduces side-effects into Elm. We definitely don't want to do the latter, so we must settle for the former.

Clipboard.js does this for us, but isn't entirely clear how at first glance.

Anyone who has tried to implement copying to the clipboard from Elm themselves, without using Clipboard.js, will likely have run into issues with it not working on Firefox. We originally tried to do this all with Elm sending a message to JavaScript world, and having some JS code that actually did the copy. This ran fine in Google Chrome, but Firefox threw an error in the console telling us that the thing doing the copying had to be getting a direct action by the user. We couldn't have a layer of indirection that sent a message on a click, and that message went to some JS that actually made the copy. That's a security faux pas.

So, how does Clipboard.js solve this problem? It attaches an event listener to the `<body>`. Whenever an event occurs, the listener checks to see if it is one of the kind that it is watching for. If so, it intercepts the call and performs the action itself, otherwise it lets the call move on down the tree.

Because the event-listener is on the body, when Elm re-renders the `div`, the listener persists.

## Thanks

This blog post would never have been written without the help of James Edward Gray II, who helped me walk through this problem and figure out why Clipboard.js was working and our solutions were not. You can find an example of working application that uses this technique [here](https://github.com/JEG2/elm_clipboard_test) and an example of this technique used in a Sinatra app with an Elm front-end [here](https://github.com/claytonflesher/oorb_sinatra_app).
