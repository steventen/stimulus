---
permalink: /handbook/origin.html
nav_prefix: Preface
order: 00
---

# The Origin of Stimulus

We write a lot of JavaScript at [Basecamp](https://basecamp.com), but we don’t use it to create “JavaScript applications” in the contemporary sense. All our applications have server-side rendered HTML at their core, then add sprinkles of JavaScript to make them sparkle.

This is the way of the [majestic monolith](https://signalvnoise.com/svn3/the-majestic-monolith/). Basecamp runs across half a dozen platforms, including native mobile apps, with a single set of controllers, views, and models created using Ruby on Rails. Having a single, shared interface that can be updated in a single place is key to being able to perform with a small team, despite the many platforms.

It allows us to party with productivity like days of yore. A throwback to when a single programmer could make rapacious progress without getting stuck in layers of indirection or distributed systems. A time before everyone thought the holy grail was to confine their server-side application to producing JSON for a JavaScript-based client application.

That’s not to say that there isn’t value in such an approach for some people, some of the time. Just that as a general approach to many applications, and certainly the likes of Basecamp, it’s a regression in overall simplicity and productivity.

And it’s also not to say that the proliferation of single-page JavaScript applications hasn’t brought real benefits. Chief amongst which has been faster, more fluid interfaces set free from the full-page refresh.

We wanted Basecamp to feel like that too. As though we had followed the herd and rewritten everything with client-side rendering or gone full-native on mobile.

This desire led us to a two-punch solution: [Turbo](https://turbo.hotwired.dev) and Stimulus.

### Turbo up high, Stimulus down low

Before I get to Stimulus, our new modest JavaScript framework, allow me to recap the proposition of Turbo.

Turbo descends from an approach called [pjax](https://github.com/defunkt/jquery-pjax), developed at GitHub. The basic concept remains the same. The reason full-page refreshes often feel slow is not so much because the browser has to process a bunch of HTML sent from a server. Browsers are really good and really fast at that. And in most cases, the fact that an HTML payload tends to be larger than a JSON payload doesn’t matter either (especially with gzipping). No, the reason is that CSS and JavaScript has to be reinitialized and reapplied to the page again. Regardless of whether the files themselves are cached. This can be pretty slow if you have a fair amount of CSS and JavaScript.

To get around this reinitialization, Turbo maintains a persistent process, just like single-page applications do. But largely an invisible one. It intercepts links and loads new pages via Ajax. The server still returns fully-formed HTML documents.

This strategy alone can make most actions in most applications feel really fast (if they’re able to return server responses in 100-200ms, which is eminently possible with caching). For Basecamp, it sped up the page-to-page transition by ~3x. It gives the application that feel of responsiveness and fluidity that was a massive part of the appeal for single-page applications.

But Turbo alone is only half the story. The coarsely grained one. Below the grade of a full page change lies all the fine-grained fidelity within a single page. The behavior that shows and hides elements, copies content to a clipboard, adds a new todo to a list, and all the other interactions we associate with a modern web application.

Prior to Stimulus, Basecamp used a smattering of different styles and patterns to apply these sprinkles. Some code was just a pinch of jQuery, some code was a similarly sized pinch of vanilla JavaScript, and some again was larger object-oriented subsystems. They all usually worked off explicit event handling hanging off a `data-behavior` attribute.

While it was easy to add new code like this, it wasn’t a comprehensive solution, and we had too many in-house styles and patterns coexisting. That made it hard to reuse code, and it made it hard for new developers to learn a consistent approach.

### The three core concepts in Stimulus

Stimulus rolls up the best of those patterns into a modest, small framework revolving around just three main concepts: Controllers, actions, and targets.

It’s designed to read as a progressive enhancement when you look at the HTML it’s addressing. Such that you can look at a single template and know which behavior is acting upon it. Here’s an example:

```html
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

You can read that and have a pretty good idea of what’s going on. Even without knowing anything about Stimulus or looking at the controller code itself. It’s almost like pseudocode. That’s very different from reading a slice of HTML that has an external JavaScript file apply event handlers to it. It also maintains the separation of concerns that has been lost in many contemporary JavaScript frameworks.

As you can see, Stimulus doesn’t bother itself with creating the HTML. Rather, it attaches itself to an existing HTML document. The HTML is, in the majority of cases, rendered on the server either on the page load (first hit or via Turbo) or via an Ajax request that changes the DOM.

Stimulus is concerned with manipulating this existing HTML document. Sometimes that means adding a CSS class that hides an element or animates it or highlights it. Sometimes it means rearranging elements in groupings. Sometimes it means manipulating the content of an element, like when we transform UTC times that can be cached into local times that can be displayed.

There are cases where you’d want Stimulus to create new DOM elements, and you’re definitely free to do that. We might even add some sugar to make it easier in the future. But it’s the minority use case. The focus is on manipulating, not creating elements.


### How Stimulus differs from mainstream JavaScript frameworks

This makes Stimulus very different from the majority of contemporary JavaScript frameworks. Almost all are focused on turning JSON into DOM elements via a template language of some sort. Many use these frameworks to birth an empty page, which is then filled exclusively with elements created through this JSON-to-template rendering.

Stimulus also differs on the question of state. Most frameworks have ways of maintaining state within JavaScript objects, and then render HTML based on that state. Stimulus is the exact opposite. State is stored in the HTML, so that controllers can be discarded between page changes, but still reinitialize as they were when the cached HTML appears again.

It really is a remarkably different paradigm. One that I’m sure many veteran JavaScript developers who’ve been used to work with contemporary frameworks will scoff at. And hey, scoff away. If you’re happy with the complexity and effort it takes to maintain an application within the maelstrom of, say, React + Redux, then Turbo + Stimulus will not appeal to you.

If, on the other hand, you have nagging sense that what you’re working on does not warrant the intense complexity and application separation such contemporary techniques imply, then you’re likely to find refuge in our approach.


### Stimulus and related ideas were extracted from the wild

At Basecamp we’ve used this architecture across several different versions of Basecamp and other applications for years. GitHub has used a similar approach to great effect. This is not only a valid alternative to the mainstream understanding of what a “modern” web application looks like, it’s an incredibly compelling one.

In fact, it feels like the same kind of secret sauce we had at Basecamp when we developed [Ruby on Rails](https://rubyonrails.org/). The sense that contemporary mainstream approaches are needlessly convoluted, and that we can do more, faster, with far less.

Furthermore, you don’t even have to choose. Stimulus and Turbo work great in conjunction with other, heavier approaches. If 80% of your application does not warrant the big rig, consider using our two-pack punch for that. Then roll out the heavy machinery for the part of your application that can really benefit from it.

At Basecamp, we have and do use several heavier-duty approaches when the occasion calls for it. Our calendars tend to use client-side rendering. Our text editor is [Trix](https://trix-editor.org/), a fully formed text processor that wouldn’t make sense as a set of Stimulus controllers.

This set of alternative frameworks is about avoiding the heavy lifting as much as possible. To stay within the request-response paradigm for all the many, many interactions that work well with that simple model. Then reaching for the expensive tooling when there’s a call for peak fidelity.

Above all, it’s a toolkit for small teams who want to compete on fidelity and reach with much larger teams using more laborious, mainstream approaches.

Give it a go.

---

David Heinemeier Hansson
---
permalink: /handbook/introduction.html
order: 01
---

# Introduction

## About Stimulus

Stimulus is a JavaScript framework with modest ambitions. Unlike other front-end frameworks, Stimulus is designed to enhance _static_ or _server-rendered_ HTML—the "HTML you already have"—by connecting JavaScript objects to elements on the page using simple annotations.

These JavaScript objects are called _controllers_, and Stimulus continuously monitors the page waiting for HTML `data-controller` attributes to appear. For each attribute, Stimulus looks at the attribute's value to find a corresponding controller class, creates a new instance of that class, and connects it to the element.

You can think of it this way: just like the `class` attribute is a bridge connecting HTML to CSS, Stimulus's `data-controller` attribute is a bridge connecting HTML to JavaScript.

Aside from controllers, the three other major Stimulus concepts are:

* _actions_, which connect controller methods to DOM events using `data-action` attributes
* _targets_, which locate elements of significance within a controller
* _values_, which read, write, and observe data attributes on the controller's element

Stimulus's use of data attributes helps separate content from behavior in the same way CSS separates content from presentation. Further, Stimulus's conventions naturally encourage you to group related code by name.

In turn, Stimulus helps you build small, reusable controllers, giving you just enough structure to keep your code from devolving into "JavaScript soup."

## About This Book

This handbook will guide you through Stimulus's core concepts by demonstrating how to write several fully functional controllers. Each chapter builds on the one before it; from start to finish, you'll learn how to:

* print a greeting addressed to the name in a text field
* copy text from a text field to the system clipboard when a button is clicked
* navigate through a slide show with multiple slides
* fetch HTML from the server into an element on the page automatically
* set up Stimulus in your own application

Once you've completed the exercises here, you may find the [reference documentation](../reference/controllers) helpful for understanding technical details about the Stimulus API.

Let's get started!
---
permalink: /handbook/hello-stimulus.html
order: 02
---

# Hello, Stimulus

The best way to learn how Stimulus works is to build a simple controller. This chapter will show you how.

## Prerequisites

To follow along, you'll need a running copy of the [`stimulus-starter`](https://github.com/hotwired/stimulus-starter) project, which is a preconfigured blank slate for exploring Stimulus.

We recommend [remixing `stimulus-starter` on Glitch](https://glitch.com/edit/#!/import/git?url=https://github.com/hotwired/stimulus-starter.git) so you can work entirely in your browser without installing anything:

[![Remix on Glitch](https://cdn.glitch.com/2703baf2-b643-4da7-ab91-7ee2a2d00b5b%2Fremix-button.svg)](https://glitch.com/edit/#!/import/git?url=https://github.com/hotwired/stimulus-starter.git)

Or, if you'd prefer to work from the comfort of your own text editor, you'll need to clone and set up `stimulus-starter`:

```
$ git clone https://github.com/hotwired/stimulus-starter.git
$ cd stimulus-starter
$ yarn install
$ yarn start
```

Then visit http://localhost:9000/ in your browser.

(Note that the `stimulus-starter` project uses the [Yarn package manager](https://yarnpkg.com/) for dependency management, so make sure you have that installed first.)

## It All Starts With HTML

Let's begin with a simple exercise using a text field and a button. When you click the button, we'll display the value of the text field in the console.

Every Stimulus project starts with HTML. Open `public/index.html` and add the following markup just after the opening `<body>` tag:

```html
<div>
  <input type="text">
  <button>Greet</button>
</div>
```

Reload the page in your browser and you should see the text field and button.

## Controllers Bring HTML to Life

At its core, Stimulus's purpose is to automatically connect DOM elements to JavaScript objects. Those objects are called _controllers_.

Let's create our first controller by extending the framework's built-in `Controller` class. Create a new file named `hello_controller.js` in the `src/controllers/` folder. Then place the following code inside:

```js
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
}
```

## Identifiers Link Controllers With the DOM

Next, we need to tell Stimulus how this controller should be connected to our HTML. We do this by placing an _identifier_ in the `data-controller` attribute on our `<div>`:

```html
<div data-controller="hello">
  <input type="text">
  <button>Greet</button>
</div>
```

Identifiers serve as the link between elements and controllers. In this case, the identifier `hello` tells Stimulus to create an instance of the controller class in `hello_controller.js`. You can learn more about how automatic controller loading works in the [Installation Guide](/handbook/installing).

## Is This Thing On?

Reload the page in your browser and you'll see that nothing has changed. How do we know whether our controller is working or not?

One way is to put a log statement in the `connect()` method, which Stimulus calls each time a controller is connected to the document.

Implement the `connect()` method in `hello_controller.js` as follows:
```js
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

Reload the page again and open the developer console. You should see `Hello, Stimulus!` followed by a representation of our `<div>`.

## Actions Respond to DOM Events

Now let's see how to change the code so our log message appears when we click the "Greet" button instead.

Start by renaming `connect()` to `greet()`:

```js
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  greet() {
    console.log("Hello, Stimulus!", this.element)
  }
}
```

We want to call the `greet()` method when the button's `click` event is triggered. In Stimulus, controller methods which handle events are called _action methods_.

To connect our action method to the button's `click` event, open `public/index.html` and add a `data-action` attribute to the button:

```html
<div data-controller="hello">
  <input type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

> ### Action Descriptors Explained
>
> The `data-action` value `click->hello#greet` is called an _action descriptor_. This particular descriptor says:
> * `click` is the event name
> * `hello` is the controller identifier
> * `greet` is the name of the method to invoke

Load the page in your browser and open the developer console. You should see the log message appear when you click the "Greet" button.

## Targets Map Important Elements To Controller Properties

We'll finish the exercise by changing our action to say hello to whatever name we've typed in the text field.

In order to do that, first we need a reference to the input element inside our controller. Then we can read the `value` property to get its contents.

Stimulus lets us mark important elements as _targets_ so we can easily reference them in the controller through corresponding properties. Open `public/index.html` and add a `data-hello-target` attribute to the input element:

```html
<div data-controller="hello">
  <input data-hello-target="name" type="text">
  <button data-action="click->hello#greet">Greet</button>
</div>
```

Next, we'll create a property for the target by adding `name` to our controller's list of target definitions. Stimulus will automatically create a `this.nameTarget` property which returns the first matching target element. We can use this property to read the element's `value` and build our greeting string.

Let's try it out. Open `hello_controller.js` and update it like so:

```js
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "name" ]

  greet() {
    const element = this.nameTarget
    const name = element.value
    console.log(`Hello, ${name}!`)
  }
}
```

Then reload the page in your browser and open the developer console. Enter your name in the input field and click the "Greet" button. Hello, world!

## Controllers Simplify Refactoring

We've seen that Stimulus controllers are instances of JavaScript classes whose methods can act as event handlers.

That means we have an arsenal of standard refactoring techniques at our disposal. For example, we can clean up our `greet()` method by extracting a `name` getter:

```js
// src/controllers/hello_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "name" ]

  greet() {
    console.log(`Hello, ${this.name}!`)
  }

  get name() {
    return this.nameTarget.value
  }
}
```

## Wrap-Up and Next Steps

Congratulations—you've just written your first Stimulus controller!

We've covered the framework's most important concepts: controllers, actions, and targets. In the next chapter, we'll see how to put those together to build a real-life controller taken right from Basecamp.
---
permalink: /handbook/building-something-real.html
order: 03
---

# Building Something Real

We've implemented our first controller and learned how Stimulus connects HTML to JavaScript. Now let's take a look at something we can use in a real application by recreating a controller from Basecamp.

## Wrapping the DOM Clipboard API

Scattered throughout Basecamp's UI are buttons like these:

<img src="../../assets/bc3-clipboard-ui.png" width="1023" height="317" class="docs__screenshot" alt="Screenshot showing a text field with an email address inside and a ”Copy to clipboard“ button to the right">

When you click one of these buttons, Basecamp copies a bit of text, such as a URL or an email address, to your clipboard.

The web platform has [an API for accessing the system clipboard](https://www.w3.org/TR/clipboard-apis/), but there's no HTML element that does what we need. To implement a "Copy to clipboard" button, we must use JavaScript.

## Implementing a Copy Button

Let's say we have an app which allows us to grant someone else access by generating a PIN for them. It would be convenient if we could display that generated PIN alongside a button to copy it to the clipboard for easy sharing.

Open `public/index.html` and replace the contents of `<body>` with a rough sketch of the button:

```html
<div>
  PIN: <input type="text" value="1234" readonly>
  <button>Copy to Clipboard</button>
</div>
```

## Setting Up the Controller

Next, create `src/controllers/clipboard_controller.js` and add an empty method `copy()`:

```js
// src/controllers/clipboard_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  copy() {
  }
}
```

Then add `data-controller="clipboard"` to the outer `<div>`. Any time this attribute appears on an element, Stimulus will connect an instance of our controller:

```html
<div data-controller="clipboard">
```

## Defining the Target

We'll need a reference to the text field so we can select its contents before invoking the clipboard API. Add `data-clipboard-target="source"` to the text field:

```html
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
```

Now add a target definition to the controller so we can access the text field element as `this.sourceTarget`:

```js
export default class extends Controller {
  static targets = [ "source" ]

  // ...
}
```

> ### What's With That `static targets` Line?
>
> When Stimulus loads your controller class, it looks for target name strings in a static array called `targets`. For each target name in the array, Stimulus adds three new properties to your controller. Here, our `"source"` target name becomes the following properties:
>
> * `this.sourceTarget` evaluates to the first `source` target in your controller's scope. If there is no `source` target, accessing the property throws an error.
> * `this.sourceTargets` evaluates to an array of all `source` targets in the controller's scope.
> * `this.hasSourceTarget` evaluates to `true` if there is a `source` target or `false` if not.
>
> You can read more about targets in the [reference documentation](/reference/targets).

## Connecting the Action

Now we're ready to hook up the Copy button.

We want a click on the button to invoke the `copy()` method in our controller, so we'll add `data-action="clipboard#copy"`:

```html
  <button data-action="clipboard#copy">Copy to Clipboard</button>
```

> ### Common Events Have a Shorthand Action Notation
>
> You might have noticed we've omitted `click->` from the action descriptor. That's because Stimulus defines `click` as the default event for actions on `<button>` elements.
>
> Certain other elements have default events, too. Here's the full list:
>
> | Element           | Default Event |
> | ----------------- | ------------- |
> | a                 | click         |
> | button            | click         |
> | details           | toggle        |
> | form              | submit        |
> | input             | input         |
> | input type=submit | click         |
> | select            | change        |
> | textarea          | input         |

Finally, in our `copy()` method, we can select the input field's contents and call the clipboard API:

```js
  copy() {
    navigator.clipboard.writeText(this.sourceTarget.value)
  }
```

Load the page in your browser and click the Copy button. Then switch back to your text editor and paste. You should see the PIN `1234`.

## Stimulus Controllers are Reusable

So far we've seen what happens when there's one instance of a controller on the page at a time.

It's not unusual to have multiple instances of a controller on the page simultaneously. For example, we might want to display a list of PINs, each with its own Copy button.

Our controller is reusable: any time we want to provide a way to copy a bit of text to the clipboard, all we need is markup on the page with the right annotations.

Let's go ahead and add another PIN to the page. Copy and paste the `<div>` so there are two identical PIN fields, then change the `value` attribute of the second:

```html
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="source" type="text" value="3737" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

Reload the page and confirm that both buttons work.

## Actions and Targets Can Go on Any Kind of Element

Now let's add one more PIN field. This time we'll use a Copy _link_ instead of a button:

```html
<div data-controller="clipboard">
  PIN: <input data-clipboard-target="source" type="text" value="3737" readonly>
  <a href="#" data-action="clipboard#copy">Copy to Clipboard</a>
</div>
```

Stimulus lets us use any kind of element we want as long as it has an appropriate `data-action` attribute.

Note that in this case, clicking the link will also cause the browser to follow the link's `href`. We can cancel this default behavior by calling `event.preventDefault()` in the action:

```js
  copy(event) {
    event.preventDefault()
    navigator.clipboard.writeText(this.sourceTarget.value)
  }
```

Similarly, our `source` target need not be an `<input type="text">`. The controller only expects it to have a `value` property and a `copy()` method. That means we can use a `<textarea>` instead:

```html
  PIN: <textarea data-clipboard-target="source" readonly>3737</textarea>
```

## Wrap-Up and Next Steps

In this chapter we looked at a real-life example of wrapping a browser API in a Stimulus controller. We saw how multiple instances of the controller can appear on the page at once, and we explored how actions and targets keep your HTML and JavaScript loosely coupled.

Now let's see how small changes to the controller's design can lead us to a more robust implementation.
---
permalink: /handbook/designing-for-resilience.html
order: 04
---

# Designing For Resilience

Although the clipboard API is [well-supported in current browsers](https://caniuse.com/#feat=clipboard), we might still expect to have a small number of people with older browsers using our application.

We should also expect people to have problems accessing our application from time to time. For example, intermittent network connectivity or CDN availability could prevent some or all of our JavaScript from loading.

It's tempting to write off support for older browsers as not worth the effort, or to dismiss network issues as temporary glitches that resolve themselves after a refresh. But often it's trivially easy to build features in a way that's gracefully resilient to these types of problems.

This resilient approach, commonly known as _progressive enhancement_, is the practice of delivering web interfaces such that the basic functionality is implemented in HTML and CSS, and tiered upgrades to that base experience are layered on top with CSS and JavaScript, progressively, when their underlying technologies are supported by the browser.

## Progressively Enhancing the PIN Field

Let's look at how we can progressively enhance our PIN field so that the Copy button is invisible unless it's supported by the browser. That way we can avoid showing someone a button that doesn't work.

We'll start by hiding the Copy button in CSS. Then we'll _feature-test_ support for the Clipboard API in our Stimulus controller. If the API is supported, we'll add a class name to the controller element to reveal the button.

We start off by adding `data-clipboard-supported-class="clipboard--supported"` to the `div` element that has the `data-controller` attribute:

```html
  <div data-controller="clipboard" data-clipboard-supported-class="clipboard--supported">
```

Then add `class="clipboard-button"` to the button element:

```html
  <button data-action="clipboard#copy" class="clipboard-button">Copy to Clipboard</button>
```

Then add the following styles to `public/main.css`:

```css
.clipboard-button {
  display: none;
}

.clipboard--supported .clipboard-button {
  display: initial;
}
```

First we'll add the `data-clipboard-supported-class` attribute inside the controller as a static class:

```js
  static classes = [ "supported" ]
```

This will let us control the specific CSS class in the HTML, so our controller becomes even more easily adaptable to different CSS approaches. The specific class added like this can be accessed via `this.supportedClass`.

Now add a `connect()` method to the controller which will test to see if the clipboard API is supported and add a class name to the controller's element:

```js
  connect() {
    if ("clipboard" in navigator) {
      this.element.classList.add(this.supportedClass);
    }
  }
```

You can place this method anywhere in the controller's class body.

If you wish, disable JavaScript in your browser, reload the page, and notice the Copy button is no longer visible.

We have progressively enhanced the PIN field: its Copy button's baseline state is hidden, becoming visible only when our JavaScript detects support for the clipboard API.

## Wrap-Up and Next Steps

In this chapter we gently modified our clipboard controller to be resilient against older browsers and degraded network conditions.

Next, we'll learn about how Stimulus controllers manage state.
---
permalink: /handbook/managing-state.html
order: 05
---

# Managing State

Most contemporary frameworks encourage you to keep state in JavaScript at all times. They treat the DOM as a write-only rendering target, reconciled by client-side templates consuming JSON from the server.

Stimulus takes a different approach. A Stimulus application's state lives as attributes in the DOM; controllers themselves are largely stateless. This approach makes it possible to work with HTML from anywhere—the initial document, an Ajax request, a Turbo visit, or even another JavaScript library—and have associated controllers spring to life automatically without any explicit initialization step.

## Building a Slideshow

In the last chapter, we learned how a Stimulus controller can maintain simple state in the document by adding a class name to an element. But what do we do when we need to store a value, not just a simple flag?

We'll investigate this question by building a slideshow controller which keeps its currently selected slide index in an attribute.

As usual, we'll begin with HTML:

```html
<div data-controller="slideshow">
  <button data-action="slideshow#previous"> ← </button>
  <button data-action="slideshow#next"> → </button>

  <div data-slideshow-target="slide">🐵</div>
  <div data-slideshow-target="slide">🙈</div>
  <div data-slideshow-target="slide">🙉</div>
  <div data-slideshow-target="slide">🙊</div>
</div>
```

Each `slide` target represents a single slide in the slideshow. Our controller will be responsible for making sure only one slide is visible at a time.

Let's draft our controller. Create a new file, `src/controllers/slideshow_controller.js`, as follows:

```js
// src/controllers/slideshow_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "slide" ]

  initialize() {
    this.index = 0
    this.showCurrentSlide()
  }

  next() {
    this.index++
    this.showCurrentSlide()
  }

  previous() {
    this.index--
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.index
    })
  }
}
```

Our controller defines a method, `showCurrentSlide()`, which loops over each slide target, toggling the [`hidden` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/hidden) if its index matches.

We initialize the controller by showing the first slide, and the `next()` and `previous()` action methods advance and rewind the current slide.

> ### Lifecycle Callbacks Explained
>
> What does the `initialize()` method do? How is it different from the `connect()` method we've used before?
>
> These are Stimulus _lifecycle callback_ methods, and they're useful for setting up or tearing down associated state when your controller enters or leaves the document.
>
> Method       | Invoked by Stimulus…
> ------------ | --------------------
> initialize() | Once, when the controller is first instantiated
> connect()    | Anytime the controller is connected to the DOM
> disconnect() | Anytime the controller is disconnected from the DOM

Reload the page and confirm that the Next button advances to the next slide.

## Reading Initial State from the DOM

Notice how our controller tracks its state—the currently selected slide—in the `this.index` property.

Now say we'd like to start one of our slideshows with the second slide visible instead of the first. How can we encode the start index in our markup?

One way might be to load the initial index with an HTML `data` attribute. For example, we could add a `data-index` attribute to the controller's element:

```html
<div data-controller="slideshow" data-index="1">
```

Then, in our `initialize()` method, we could read that attribute, convert it to an integer, and pass it to `showCurrentSlide()`:

```js
  initialize() {
    this.index = Number(this.element.dataset.index)
    this.showCurrentSlide()
  }
```

This might get the job done, but it's clunky, requires us to make a decision about what to name the attribute, and doesn't help us if we want to access the index again or increment it and persist the result in the DOM.

### Using Values

Stimulus controllers support typed value properties which automatically map to data attributes. When we add a value definition to the top of our controller class:

```js
  static values = { index: Number }
```

Stimulus will create a `this.indexValue` controller property associated with a `data-slideshow-index-value` attribute, and handle the numeric conversion for us.

Let's see that in action. Add the associated data attribute to our HTML:

```html
<div data-controller="slideshow" data-slideshow-index-value="1">
```

Then add a `static values` definition to the controller and change the `initialize()` method to log `this.indexValue`:

```js
export default class extends Controller {
  static values = { index: Number }

  initialize() {
    console.log(this.indexValue)
    console.log(typeof this.indexValue)
  }

  // …
}
```

Reload the page and verify that the console shows `1` and `Number`.

> ### What's with that `static values` line?
>
> Similar to targets, you define values in a Stimulus controller by describing them in a static object property called `values`. In this case, we've defined a single numeric value called `index`. You can read more about value definitions in the [reference documentation](/reference/values).

Now let's update `initialize()` and the other methods in the controller to use `this.indexValue` instead of `this.index`. Here's how the controller should look when we're done:

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "slide" ]
  static values = { index: Number }

  initialize() {
    this.showCurrentSlide()
  }

  next() {
    this.indexValue++
    this.showCurrentSlide()
  }

  previous() {
    this.indexValue--
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.indexValue
    })
  }
}
```

Reload the page and use the web inspector to confirm the controller element's `data-slideshow-index-value` attribute changes as you move from one slide to the next.

### Change Callbacks

Our revised controller improves on the original version, but the repeated calls to `this.showCurrentSlide()` stand out. We have to manually update the state of the document when the controller initializes and after every place where we update `this.indexValue`.

We can define a Stimulus value change callback to clean up the repetition and specify how the controller should respond whenever the index value changes.

First, remove the `initialize()` method and define a new method, `indexValueChanged()`. Then remove the calls to `this.showCurrentSlide()` from `next()` and `previous()`:

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "slide" ]
  static values = { index: Number }

  next() {
    this.indexValue++
  }

  previous() {
    this.indexValue--
  }

  indexValueChanged() {
    this.showCurrentSlide()
  }

  showCurrentSlide() {
    this.slideTargets.forEach((element, index) => {
      element.hidden = index !== this.indexValue
    })
  }
}
```

Reload the page and confirm the slideshow behavior is the same.

Stimulus calls the `indexValueChanged()` method at initialization and in response to any change to the `data-slideshow-index-value` attribute. You can even fiddle with the attribute in the web inspector and the controller will change slides in response. Go ahead—try it out!

### Setting Defaults

It's also possible to set a default values as part of the static definition. This is done like so:

```js
  static values = { index: { type: Number, default: 2 } }
```

That would start the index at 2, if no `data-slideshow-index-value` attribute was defined on the controller element. If you had other values, you can mix and match what needs a default and what doesn't:

```js
  static values = { index: Number, effect: { type: String, default: "kenburns" } }
```

## Wrap-Up and Next Steps

In this chapter we've seen how to use the values to load and persist the current index of a slideshow controller.

From a usability perspective, our controller is incomplete. The Previous button appears to do nothing when you are looking at the first slide. Internally, `indexValue` decrements from `0` to `-1`. Could we make the value wrap around to the _last_ slide index instead? (There's a similar problem with the Next button.)

Next we'll look at how to keep track of external resources, such as timers and HTTP requests, in Stimulus controllers.
---
permalink: /handbook/working-with-external-resources.html
order: 06
---

# Working With External Resources

In the last chapter we learned how to load and persist a controller's internal state using values.

Sometimes our controllers need to track the state of external resources, where by _external_ we mean anything that isn't in the DOM or a part of Stimulus. For example, we may need to issue an HTTP request and respond as the request's state changes. Or we may want to start a timer and then stop it when the controller is no longer connected. In this chapter we'll see how to do both of those things.

## Asynchronously Loading HTML

Let's learn how to populate parts of a page asynchronously by loading and inserting remote fragments of HTML. We use this technique in Basecamp to keep our initial page loads fast, and to keep our views free of user-specific content so they can be cached more effectively.

We'll build a general-purpose content loader controller which populates its element with HTML fetched from the server. Then we'll use it to load a list of unread messages like you'd see in an email inbox.

Begin by sketching the inbox in `public/index.html`:

```html
<div data-controller="content-loader"
     data-content-loader-url-value="/messages.html"></div>
```

Then create a new `public/messages.html` file with some HTML for our message list:

```html
<ol>
  <li>New Message: Stimulus Launch Party</li>
  <li>Overdue: Finish Stimulus 1.0</li>
</ol>
```

(In a real application you'd generate this HTML dynamically on the server, but for demonstration purposes we'll just use a static file.)

Now we can implement our controller:

```js
// src/controllers/content_loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { url: String }

  connect() {
    this.load()
  }

  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
}
```

When the controller connects, we kick off a [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) request to the URL specified in the element's `data-content-loader-url-value` attribute. Then we load the returned HTML by assigning it to our element's `innerHTML` property.

Open the network tab in your browser's developer console and reload the page. You'll see a request representing the initial page load, followed by our controller's subsequent request to `messages.html`.

## Refreshing Automatically With a Timer

Let's improve our controller by changing it to periodically refresh the inbox so it's always up-to-date.

We'll use the `data-content-loader-refresh-interval-value` attribute to specify how often the controller should reload its contents, in milliseconds:

```html
<div data-controller="content-loader"
     data-content-loader-url-value="/messages.html"
     data-content-loader-refresh-interval-value="5000"></div>
```

Now we can update the controller to check for the interval and, if present, start a refresh timer.

Add a `static values` definition to the controller, and define a new method `startRefreshing()`:

```js
export default class extends Controller {
  static values = { url: String, refreshInterval: Number }

  startRefreshing() {
    setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }

  // …
}
```

Then update the `connect()` method to call `startRefreshing()` if an interval value is present:

```js
  connect() {
    this.load()

    if (this.hasRefreshIntervalValue) {
      this.startRefreshing()
    }
  }
```

Reload the page and observe a new request once every five seconds in the developer console. Then make a change to `public/messages.html` and wait for it to appear in the inbox.

## Releasing Tracked Resources

We start our timer when the controller connects, but we never stop it. That means if our controller's element were to disappear, the controller would continue to issue HTTP requests in the background.

We can fix this issue by modifying our `startRefreshing()` method to keep a reference to the timer:

```js
  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }
```

Then we can add a corresponding `stopRefreshing()` method below to cancel the timer:

```js
  stopRefreshing() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer)
    }
  }
```

Finally, to instruct Stimulus to cancel the timer when the controller disconnects, we'll add a `disconnect()` method:

```js
  disconnect() {
    this.stopRefreshing()
  }
```

Now we can be sure a content loader controller will only issue requests when it's connected to the DOM.

Let's take a look at our final controller class:

```js
// src/controllers/content_loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { url: String, refreshInterval: Number }

  connect() {
    this.load()

    if (this.hasRefreshIntervalValue) {
      this.startRefreshing()
    }
  }

  disconnect() {
    this.stopRefreshing()
  }

  load() {
    fetch(this.urlValue)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }

  startRefreshing() {
    this.refreshTimer = setInterval(() => {
      this.load()
    }, this.refreshIntervalValue)
  }

  stopRefreshing() {
    if (this.refreshTimer) {
      clearInterval(this.refreshTimer)
    }
  }
}
```

## Using action parameters

If we wanted to make the loader work with multiple different sources, we could do it using action parameters. Take this HTML:

```html
<div data-controller="content-loader">
  <a href="#" data-content-loader-url-param="/messages.html" data-action="content-loader#load">Messages</a>
  <a href="#" data-content-loader-url-param="/comments.html" data-action="content-loader#load">Comments</a>
</div>
```

Then we can use those parameters through the `load` action:

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  load({ params }) {
    fetch(params.url)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
}
```

We could even destruct the params to just get the URL parameter:

```js
  load({ params: { url } }) {
    fetch(url)
      .then(response => response.text())
      .then(html => this.element.innerHTML = html)
  }
```

## Wrap-Up and Next Steps

In this chapter we've seen how to acquire and release external resources using Stimulus lifecycle callbacks.

Next we'll see how to install and configure Stimulus in your own application.
---
permalink: /handbook/installing.html
order: 07
---

# Installing Stimulus in Your Application

To install Stimulus in your application, add the [`@hotwired/stimulus` npm package](https://www.npmjs.com/package/@hotwired/stimulus) to your JavaScript bundle. Or, import [`stimulus.js`](https://unpkg.com/@hotwired/stimulus/dist/stimulus.js) in a `<script type="module">` tag.

## Using Stimulus for Rails

If you're using [Stimulus for Rails](https://github.com/hotwired/stimulus-rails/) together with an [import map](https://github.com/rails/importmap-rails), the integration will automatically load all controller files from `app/javascript/controllers`.

### Controller Filenames Map to Identifiers

Name your controller files `[identifier]_controller.js`, where `identifier` corresponds to each controller's `data-controller` identifier in your HTML.

Stimulus for Rails conventionally separates multiple words in filenames using underscores. Each underscore in a controller's filename translates to a dash in its identifier.

You may also namespace your controllers using subfolders. Each forward slash in a namespaced controller file's path becomes two dashes in its identifier.

If you prefer, you may use dashes instead of underscores anywhere in a controller's filename. Stimulus treats them identically.

If your controller file is named… | its identifier will be…
--------------------------------- | -----------------------
clipboard_controller.js           | clipboard
date_picker_controller.js         | date-picker
users/list_item_controller.js     | users\-\-list-item
local-time-controller.js          | local-time

## Using Webpack Helpers

If you're using Webpack as your JavaScript bundler, you can use the [@hotwired/stimulus-webpack-helpers](https://www.npmjs.com/package/@hotwired/stimulus-webpack-helpers) package to get the same form of autoloading behavior as Stimulus for Rails. First add the package, then use it like this:

```js
import { Application } from "@hotwired/stimulus"
import { definitionsFromContext } from "@hotwired/stimulus-webpack-helpers"

window.Stimulus = Application.start()
const context = require.context("./controllers", true, /\.js$/)
Stimulus.load(definitionsFromContext(context))
```

## Using Other Build Systems

Stimulus works with other build systems too, but without support for controller autoloading. Instead, you must explicitly load and register controller files with your application instance:

```js
// src/application.js
import { Application } from "@hotwired/stimulus"

import HelloController from "./controllers/hello_controller"
import ClipboardController from "./controllers/clipboard_controller"

window.Stimulus = Application.start()
Stimulus.register("hello", HelloController)
Stimulus.register("clipboard", ClipboardController)
```

If you're using stimulus-rails with a builder like esbuild, you can use the `stimulus:manifest:update` Rake task and `./bin/rails generate stimulus [controller]` generator to keep a controller index file located at `app/javascript/controllers/index.js` automatically updated.

## Using Without a Build System

If you prefer not to use a build system, you can load Stimulus in a `<script type="module">` tag:

```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <script type="module">
    import { Application, Controller } from "https://unpkg.com/@hotwired/stimulus/dist/stimulus.js"
    window.Stimulus = Application.start()

    Stimulus.register("hello", class extends Controller {
      static targets = [ "name" ]

      connect() {
      }
    })
  </script>
</head>
<body>
  <div data-controller="hello">
    <input data-hello-target="name" type="text">
    …
  </div>
</body>
</html>
```

## Overriding Attribute Defaults
In case Stimulus `data-*` attributes conflict with another library in your project, they can be overridden when creating the Stimulus `Application`.

- `data-controller`
- `data-action`
- `data-target`

These core Stimulus attributes can be overridden (see: [schema.ts](https://github.com/hotwired/stimulus/blob/main/src/core/schema.ts)):

```js
// src/application.js
import { Application, defaultSchema } from "@hotwired/stimulus"

const customSchema = {
  ...defaultSchema,
  actionAttribute: 'data-stimulus-action'
}

window.Stimulus = Application.start(document.documentElement, customSchema);
```

## Error handling

All calls from Stimulus to your application's code are wrapped in a `try ... catch` block.

If your code throws an error, it will be caught by Stimulus and logged to the browser console, including extra detail such as the controller name and event or lifecycle function being called. If you use an error tracking system that defines `window.onerror`, Stimulus will also pass the error on to it.

You can override how Stimulus handles errors by defining `Application#handleError`:

```js
// src/application.js
import { Application } from "@hotwired/stimulus"
window.Stimulus = Application.start()

Stimulus.handleError = (error, message, detail) => {
  console.warn(message, detail)
  ErrorTrackingSystem.captureException(error)
}
```

## Debugging

If you've assigned your Stimulus application to `window.Stimulus`, you can turn on [debugging mode](https://github.com/hotwired/stimulus/pull/354) from the console with `Stimulus.debug = true`. You can also set this flag when you're configuring your application instance in the source code.


## Browser Support

Stimulus supports all evergreen, self-updating desktop and mobile browsers out of the box. Stimulus 3+ does not support Internet Explorer 11 (but you can use Stimulus 2 with the @stimulus/polyfills for that).


# Reference

---
permalink: /reference/actions.html
order: 02
---

# Actions

_Actions_ are how you handle DOM events in your controllers.

<meta data-controller="callout" data-callout-text-value="click->gallery#next">

```html
<div data-controller="gallery">
  <button data-action="click->gallery#next">…</button>
</div>
```

<meta data-controller="callout" data-callout-text-value="next">

```js
// controllers/gallery_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  next(event) {
    // …
  }
}
```

An action is a connection between:

* a controller method
* the controller's element
* a DOM event listener

## Descriptors

The `data-action` value `click->gallery#next` is called an _action descriptor_. In this descriptor:

* `click` is the name of the DOM event to listen for
* `gallery` is the controller identifier
* `next` is the name of the method to invoke

### Event Shorthand

Stimulus lets you shorten the action descriptors for some common element/event pairs, such as the button/click pair above, by omitting the event name:

<meta data-controller="callout" data-callout-text-value="gallery#next">

```html
<button data-action="gallery#next">…</button>
```

The full set of these shorthand pairs is as follows:

Element           | Default Event
----------------- | -------------
a                 | click
button            | click
details           | toggle
form              | submit
input             | input
input type=submit | click
select            | change
textarea          | input


## KeyboardEvent Filter

There may be cases where [KeyboardEvent](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent) Actions should only call the Controller method when certain keystrokes are used.

You can install an event listener that responds only to the `Escape` key by adding `.esc` to the event name of the action descriptor, as in the following example.

```html
<div data-controller="modal"
     data-action="keydown.esc->modal#close" tabindex="0">
</div>
```

This will only work if the event being fired is a keyboard event.

The correspondence between these filter and keys is shown below.

Filter    | Key Name
--------  | --------
enter     | Enter
tab       | Tab
esc       | Escape
space     | " "
up        | ArrowUp
down      | ArrowDown
left      | ArrowLeft
right     | ArrowRight
home      | Home
end       | End
page_up   | PageUp
page_down | PageDown
[a-z]     | [a-z]
[0-9]     | [0-9]

If you need to support other keys, you can customize the modifiers using a custom schema.

```javascript
import { Application, defaultSchema } from "@hotwired/stimulus"

const customSchema = {
  ...defaultSchema,
  keyMappings: { ...defaultSchema.keyMappings, at: "@" },
}

const app = Application.start(document.documentElement, customSchema)
```

If you want to subscribe to a compound filter using a modifier key, you can write it like `ctrl+a`.

```html
<div data-action="keydown.ctrl+a->listbox#selectAll" role="option" tabindex="0">...</div>
```

The list of supported modifier keys is shown below.

| Modifier | Notes              |
| -------- | ------------------ |
| `alt`    | `option` on MacOS  |
| `ctrl`   |                    |
| `meta`   | Command key on MacOS |
| `shift`  |                    |

### Global Events

Sometimes a controller needs to listen for events dispatched on the global `window` or `document` objects.

You can append `@window` or `@document` to the event name (along with any filter modifier) in an action descriptor to install the event listener on `window` or `document`, respectively, as in the following example:

<meta data-controller="callout" data-callout-text-value="resize@window">

```html
<div data-controller="gallery"
     data-action="resize@window->gallery#layout">
</div>
```

### Options

You can append one or more _action options_ to an action descriptor if you need to specify [DOM event listener options](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

<meta data-controller="callout" data-callout-text-value=":!passive">
<meta data-controller="callout" data-callout-text-value=":capture">

```html
<div data-controller="gallery"
     data-action="scroll->gallery#layout:!passive">
  <img data-action="click->gallery#open:capture">
```

Stimulus supports the following action options:

Action option | DOM event listener option
------------- | -------------------------
`:capture`    | `{ capture: true }`
`:once`       | `{ once: true }`
`:passive`    | `{ passive: true }`
`:!passive`   | `{ passive: false }`

On top of that, Stimulus also supports the following action options which are not natively supported by the DOM event listener options:

Custom action option | Description
-------------------- | -----------
`:stop`              | calls `.stopPropagation()` on the event before invoking the method
`:prevent`           | calls `.preventDefault()` on the event before invoking the method
`:self`              | only invokes the method if the event was fired by the element itself

You can register your own action options with the `Application.registerActionOption` method.

For example, consider that a `<details>` element will dispatch a [toggle][]
event whenever it's toggled. A custom `:open` action option would help
to route events whenever the element is toggled _open_:

```javascript
import { Application } from "@hotwired/stimulus"

const application = Application.start()

application.registerActionOption("open", ({ event }) => {
  if (event.type == "toggle") {
    return event.target.open == true
  } else {
    return true
  }
})
```

Similarly, a custom `:!open` action option could route events whenever the
element is toggled _closed_. Declaring the action descriptor option with a `!`
prefix will yield a `value` argument set to `false` in the callback:

```javascript
import { Application } from "@hotwired/stimulus"

const application = Application.start()

application.registerActionOption("open", ({ event, value }) => {
  if (event.type == "toggle") {
    return event.target.open == value
  } else {
    return true
  }
})
```

In order to prevent the event from being routed to the controller action, the
`registerActionOption` callback function must return `false`. Otherwise, to
route the event to the controller action, return `true`.

The callback accepts a single object argument with the following keys:

| Name       | Description                                                                                           |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| name       | String: The option's name (`"open"` in the example above)                                             |
| value      | Boolean: The value of the option (`:open` would yield `true`, `:!open` would yield `false`)           |
| event      | [Event][]: The event instance, including with the `params` action parameters on the submitter element |
| element    | [Element]: The element where the action descriptor is declared                                        |
| controller | The `Controller` instance which would receive the method call                                         |

[toggle]: https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement/toggle_event
[Event]: https://developer.mozilla.org/en-US/docs/web/api/event
[Element]: https://developer.mozilla.org/en-US/docs/Web/API/element

## Event Objects

An _action method_ is the method in a controller which serves as an action's event listener.

The first argument to an action method is the DOM _event object_. You may want access to the event for a number of reasons, including:

* to read the key code from a keyboard event
* to read the coordinates of a mouse event
* to read data from an input event
* to read params from the action submitter element
* to prevent the browser's default behavior for an event
* to find out which element dispatched an event before it bubbled up to this action

The following basic properties are common to all events:

Event Property      | Value
------------------- | -----
event.type          | The name of the event (e.g. `"click"`)
event.target        | The target that dispatched the event (i.e. the innermost element that was clicked)
event.currentTarget | The target on which the event listener is installed (either the element with the `data-action` attribute, or `document` or `window`)
event.params        | The action params passed by the action submitter element

<br>The following event methods give you more control over how events are handled:

Event Method            | Result
----------------------- | ------
event.preventDefault()  | Cancels the event's default behavior (e.g. following a link or submitting a form)
event.stopPropagation() | Stops the event before it bubbles up to other listeners on parent elements

## Multiple Actions

The `data-action` attribute's value is a space-separated list of action descriptors.

It's common for any given element to have many actions. For example, the following input element calls a `field` controller's `highlight()` method when it gains focus, and a `search` controller's `update()` method every time the element's value changes:

<meta data-controller="callout" data-callout-text-value="focus->field#highlight">
<meta data-controller="callout" data-callout-text-value="input->search#update">

```html
<input type="text" data-action="focus->field#highlight input->search#update">
```

When an element has more than one action for the same event, Stimulus invokes the actions from left to right in the order that their descriptors appear.

The action chain can be stopped at any point by calling `Event#stopImmediatePropagation()` within an action. Any additional actions to the right will be ignored:

```javascript
highlight(event) {
  event.stopImmediatePropagation()
  // ...
}
```

## Naming Conventions

Always use camelCase to specify action names, since they map directly to methods on your controller.

Avoid action names that simply repeat the event's name, such as `click`, `onClick`, or `handleClick`:

<meta data-controller="callout" data-callout-text-value="#click" data-callout-type-value="avoid">

```html
<button data-action="click->profile#click">Don't</button>
```

Instead, name your action methods based on what will happen when they're called:

<meta data-controller="callout" data-callout-text-value="#showDialog" data-callout-type-value="prefer">

```html
<button data-action="click->profile#showDialog">Do</button>
```

This will help you reason about the behavior of a block of HTML without having to look at the controller source.

## Action Parameters

Actions can have parameters that are be passed from the submitter element. They follow the format of `data-[identifier]-[param-name]-param`. Parameters must be specified on the same element as the action they intend to be passed to is declared.

All parameters are automatically typecast to either a `Number`, `String`, `Object`, or `Boolean`, inferred by their value:

Data attribute                                  | Param                | Type
----------------------------------------------- | -------------------- | --------
`data-item-id-param="12345"`                    | `12345`              | Number
`data-item-url-param="/votes"`                  | `"/votes"`           | String
`data-item-payload-param='{"value":"1234567"}'` | `{ value: 1234567 }` | Object
`data-item-active-param="true"`                 | `true`               | Boolean


<br>Consider this setup:

```html
<div data-controller="item spinner">
  <button data-action="item#upvote spinner#start" 
    data-item-id-param="12345" 
    data-item-url-param="/votes"
    data-item-payload-param='{"value":"1234567"}' 
    data-item-active-param="true">…</button>
</div>
```

It will call both `ItemController#upvote` and `SpinnerController#start`, but only the former will have any parameters passed to it:

```js
// ItemController
upvote(event) {
  // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
  console.log(event.params) 
}

// SpinnerController
start(event) {
  // {}
  console.log(event.params) 
}
```

If we don't need anything else from the event, we can destruct the params:

```js
upvote({ params }) {
  // { id: 12345, url: "/votes", active: true, payload: { value: 1234567 } }
  console.log(params) 
}
```

Or destruct only the params we need, in case multiple actions on the same controller share the same submitter element:

```js
upvote({ params: { id, url } }) {
  console.log(id) // 12345
  console.log(url) // "/votes"
}
```
---
permalink: /reference/controllers.html
order: 00
---

# Controllers

A _controller_ is the basic organizational unit of a Stimulus application.

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  // …
}
```

Controllers are instances of JavaScript classes that you define in your application. Each controller class inherits from the `Controller` base class exported by the `@hotwired/stimulus` module.

## Properties

Every controller belongs to a Stimulus `Application` instance and is associated with an HTML element. Within a controller class, you can access the controller's:

* application, via the `this.application` property
* HTML element, via the `this.element` property
* identifier, via the `this.identifier` property

## Modules

Define your controller classes in JavaScript modules, one per file. Export each controller class as the module's default object, as in the example above.

Place these modules in the `controllers/` directory. Name the files `[identifier]_controller.js`, where `[identifier]` corresponds to each controller's identifier.

## Identifiers

An _identifier_ is the name you use to reference a controller class in HTML.

When you add a `data-controller` attribute to an element, Stimulus reads the identifier from the attribute's value and creates a new instance of the corresponding controller class.

For example, this element has a controller which is an instance of the class defined in `controllers/reference_controller.js`:

<meta data-controller="callout" data-callout-text-value="reference">

```html
<div data-controller="reference"></div>
```

The following is an example of how Stimulus will generate identifiers for controllers in its require context:

If your controller file is named… | its identifier will be…
--------------------------------- | -----------------------
clipboard_controller.js           | clipboard
date_picker_controller.js         | date-picker
users/list_item_controller.js     | users\-\-list-item
local-time-controller.js          | local-time

## Scopes

When Stimulus connects a controller to an element, that element and all of its children make up the controller's _scope_.

For example, the `<div>` and `<h1>` below are part of the controller's scope, but the surrounding `<main>` element is not.

```html
<main>
  <div data-controller="reference">
    <h1>Reference</h1>
  </div>
</main>
```

## Nested Scopes

When nested, each controller is only aware of its own scope excluding the scope of any controllers nested within.

For example, the `#parent` controller below is only aware of the `item` targets directly within its scope, but not any targets of the `#child` controller.

```html
<ul id="parent" data-controller="list">
  <li data-list-target="item">One</li>
  <li data-list-target="item">Two</li>
  <li>
    <ul id="child" data-controller="list">
      <li data-list-target="item">I am</li>
      <li data-list-target="item">a nested list</li>
    </ul>
  </li>
</ul>
```

## Multiple Controllers

The `data-controller` attribute's value is a space-separated list of identifiers:

<meta data-controller="callout" data-callout-text-value="clipboard">
<meta data-controller="callout" data-callout-text-value="list-item">

```html
<div data-controller="clipboard list-item"></div>
```

It's common for any given element on the page to have many controllers. In the example above, the `<div>` has two connected controllers, `clipboard` and `list-item`.

Similarly, it's common for multiple elements on the page to reference the same controller class:

<meta data-controller="callout" data-callout-text-value="list-item">

```html
<ul>
  <li data-controller="list-item">One</li>
  <li data-controller="list-item">Two</li>
  <li data-controller="list-item">Three</li>
</ul>
```

Here, each `<li>` has its own instance of the `list-item` controller.

## Naming Conventions

Always use camelCase for method and property names in a controller class.

When an identifier is composed of more than one word, write the words in kebab-case (i.e., by using dashes: `date-picker`, `list-item`).

In filenames, separate multiple words using either underscores or dashes (snake_case or kebab-case: `controllers/date_picker_controller.js`, `controllers/list-item-controller.js`).

## Registration

If you use Stimulus for Rails with an import map or Webpack together with the `@hotwired/stimulus-webpack-helpers` package, your application will automatically load and register controller classes following the conventions above.

If not, your application must manually load and register each controller class.

### Registering Controllers Manually

To manually register a controller class with an identifier, first import the class, then call the `Application#register` method on your application object:

```js
import ReferenceController from "./controllers/reference_controller"

application.register("reference", ReferenceController)
```

You can also register a controller class inline instead of importing it from a module:

```js
import { Controller } from "@hotwired/stimulus"

application.register("reference", class extends Controller {
  // …
})
```

### Preventing Registration Based On Environmental Factors

If you only want a controller registered and loaded if certain environmental factors are met – such a given user agent – you can overwrite the static `shouldLoad` method:

```js
class UnloadableController extends ApplicationController {
  static get shouldLoad() {
    return false
  }
}

// This controller will not be loaded
application.register("unloadable", UnloadableController)
```

### Trigger Behaviour When A Controller Is Registered

If you want to trigger some behaviour once a controller has been registered you can add a static `afterLoad` method:

```js
class SpinnerButton extends Controller {
  static legacySelector = ".legacy-spinner-button"

  static afterLoad(identifier, application) {
    // use the application instance to read the configured 'data-controller' attribute
    const { controllerAttribute } = application.schema

    // update any legacy buttons with the controller's registered identifier
    const updateLegacySpinners = () => {
      document.querySelector(this.legacySelector).forEach((element) => {
        element.setAttribute(controllerAttribute, identifier)
      })
    }

    // called as soon as registered so DOM may not have loaded yet
    if (document.readyState == "loading") {
      document.addEventListener("DOMContentLoaded", updateLegacySpinners)
    } else {
      updateLegacySpinners()
    }
  }
}

// This controller will update any legacy spinner buttons to use the controller
application.register("spinner-button", SpinnerButton)
```

The `afterLoad` method will get called as soon as the controller has been registered, even if no controlled elements exist in the DOM. The function will be called bound to the original controller constructor along with two arguments; the `identifier` that was used when registering the controller and the Stimulus application instance.

## Cross-Controller Coordination With Events

If you need controllers to communicate with each other, you should use events. The `Controller` class has a convenience method called `dispatch` that makes this easier. It takes an `eventName` as the first argument, which is then automatically prefixed with the name of the controller separated by a colon. The payload is held in `detail`. It works like this:

```js
class ClipboardController extends Controller {
  static targets = [ "source" ]

  copy() {
    this.dispatch("copy", { detail: { content: this.sourceTarget.value } })
    navigator.clipboard.writeText(this.sourceTarget.value)
  }
}
```

And this event can then be routed to an action on another controller:

```html
<div data-controller="clipboard effects" data-action="clipboard:copy->effects#flash">
  PIN: <input data-clipboard-target="source" type="text" value="1234" readonly>
  <button data-action="clipboard#copy">Copy to Clipboard</button>
</div>
```

So when the `Clipboard#copy` action is invoked, the `Effects#flash` action will be too:

```js
class EffectsController extends Controller {
  flash({ detail: { content } }) {
    console.log(content) // 1234
  }
}
```

If the two controllers don't belong to the same HTML element, the `data-action` attribute
needs to be added to the *receiving* controller's element. And if the receiving controller's
element is not a parent (or same) of the emitting controller's element, you need to add
`@window` to the event:

```html
<div data-action="clipboard:copy@window->effects#flash">
```

`dispatch` accepts additional options as the second parameter as follows:

option       | default            | notes
-------------|--------------------|----------------------------------------------------------------------------------------------
`detail`     | `{}` empty object  | See [CustomEvent.detail](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent/detail)
`target`     | `this.element`     | See [Event.target](https://developer.mozilla.org/en-US/docs/Web/API/Event/target)
`prefix`     | `this.identifier`  | If the prefix is falsey (e.g. `null` or `false`), only the `eventName` will be used. If you provide a string value the `eventName` will be prepended with the provided string and a colon. 
`bubbles`    | `true`             | See [Event.bubbles](https://developer.mozilla.org/en-US/docs/Web/API/Event/bubbles)
`cancelable` | `true`             | See [Event.cancelable](https://developer.mozilla.org/en-US/docs/Web/API/Event/cancelable)

`dispatch` will return the generated [`CustomEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent), you can use this to provide a way for the event to be cancelled by any other listeners as follows:

```js
class ClipboardController extends Controller {
  static targets = [ "source" ]

  copy() {
    const event = this.dispatch("copy", { cancelable: true })
    if (event.defaultPrevented) return
    navigator.clipboard.writeText(this.sourceTarget.value)
  }
}
```

```js
class EffectsController extends Controller {
  flash(event) {
    // this will prevent the default behaviour as determined by the dispatched event
    event.preventDefault()
  }
}
```

## Directly Invoking Other Controllers

If for some reason it is not possible to use events to communicate between controllers, you can reach a controller instance via the `getControllerForElementAndIdentifier` method from the application. This should only be used if you have a unique problem that cannot be solved through the more general way of using events, but if you must, this is how:

```js
class MyController extends Controller {
  static targets = [ "other" ]

  copy() {
    const otherController = this.application.getControllerForElementAndIdentifier(this.otherTarget, 'other')
    otherController.otherMethod()
  }
}
```
---
permalink: /reference/css-classes.html
order: 06
---

# CSS Classes

In HTML, a _CSS class_ defines a set of styles which can be applied to elements using the `class` attribute.

CSS classes are a convenient tool for changing styles and playing animations programmatically. For example, a Stimulus controller might add a "loading" class to an element when it is performing an operation in the background, and then style that class in CSS to display a progress indicator:

```html
<form data-controller="search" class="search--busy">
```

```css
.search--busy {
  background-image: url(throbber.svg) no-repeat;
}
```

As an alternative to hard-coding classes with JavaScript strings, Stimulus lets you refer to CSS classes by _logical name_ using a combination of data attributes and controller properties.

## Definitions

Define CSS classes by logical name in your controller using the `static classes` array:

<meta data-controller="callout" data-callout-text-value="static classes = [ &quot;loading&quot; ]">

```js
// controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static classes = [ "loading" ]

  // …
}
```

## Attributes

The logical names defined in the controller's `static classes` array map to _CSS class attributes_ on the controller's element.

<meta data-controller="callout" data-callout-text-value="data-search-loading-class=&quot;search--busy&quot;">

```html
<form data-controller="search"
      data-search-loading-class="search--busy">
  <input data-action="search#loadResults">
</form>
```

Construct a CSS class attribute by joining together the controller identifier and logical name in the format `data-[identifier]-[logical-name]-class`. The attribute's value can be a single CSS class name or a list of multiple class names.

**Note:** CSS class attributes must be specified on the same element as the `data-controller` attribute.

If you want to specify multiple CSS classes for a logical name, separate the classes with spaces:

<meta data-controller="callout" data-callout-text-value="data-search-loading-class=&quot;bg-gray-500 animate-spinner cursor-busy&quot;">

```html
<form data-controller="search"
      data-search-loading-class="bg-gray-500 animate-spinner cursor-busy">
  <input data-action="search#loadResults">
</form>
```

## Properties

For each logical name defined in the `static classes` array, Stimulus adds the following _CSS class properties_ to your controller:

Kind        | Name                         | Value
----------- | ---------------------------- | -----
Singular    | `this.[logicalName]Class`    | The value of the CSS class attribute corresponding to `logicalName`
Plural      | `this.[logicalName]Classes`  | An array of all classes in the corresponding CSS class attribute, split by spaces
Existential | `this.has[LogicalName]Class` | A boolean indicating whether or not the CSS class attribute is present

<br>Use these properties to apply CSS classes to elements with the `add()` and `remove()` methods of the [DOM `classList` API](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList).

For example, to display a loading indicator on the `search` controller's element before fetching results, you might implement the `loadResults` action like so:

<meta data-controller="callout" data-callout-text-value="this.loadingClass">

```js
export default class extends Controller {
  static classes = [ "loading" ]

  loadResults() {
    this.element.classList.add(this.loadingClass)

    fetch(/* … */)
  }
}
```

If a CSS class attribute contains a list of class names, its singular CSS class property returns the first class in the list.

Use the plural CSS class property to access all class names as an array. Combine this with [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) to apply multiple classes at once:

<meta data-controller="callout" data-callout-text-value="...this.loadingClasses">

```js
export default class extends Controller {
  static classes = [ "loading" ]

  loadResults() {
    this.element.classList.add(...this.loadingClasses)

    fetch(/* … */)
  }
}
```

**Note:** Stimulus will throw an error if you attempt to access a CSS class property when a matching CSS class attribute is not present.

## Naming Conventions

Use camelCase to specify logical names in CSS class definitions. Logical names map to camelCase CSS class properties:

<meta data-controller="callout" data-callout-text-value="noResultsClass">
<meta data-controller="callout" data-callout-text-value="noResults">

```js
export default class extends Controller {
  static classes = [ "loading", "noResults" ]

  loadResults() {
    // …
    if (results.length == 0) {
      this.element.classList.add(this.noResultsClass)
    }
  }
}
```

In HTML, write CSS class attributes in kebab-case:

<meta data-controller="callout" data-callout-text-value="no-results">

```html
<form data-controller="search"
      data-search-loading-class="search--busy"
      data-search-no-results-class="search--empty">
```

When constructing CSS class attributes, follow the conventions for identifiers as described in [Controllers: Naming Conventions](controllers#naming-conventions).
---
permalink: /reference/lifecycle-callbacks.html
order: 01
---

# Lifecycle Callbacks

Special methods called _lifecycle callbacks_ allow you to respond whenever a controller or certain targets connects to and disconnects from the document.

<meta data-controller="callout" data-callout-text-value="connect()">

```js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    // …
  }
}
```

## Methods

You may define any of the following methods in your controller:

Method       | Invoked by Stimulus…
------------ | --------------------
initialize() | Once, when the controller is first instantiated
[name]TargetConnected(target: Element) | Anytime a target is connected to the DOM
connect()    | Anytime the controller is connected to the DOM
[name]TargetDisconnected(target: Element) | Anytime a target is disconnected from the DOM
disconnect() | Anytime the controller is disconnected from the DOM

## Connection

A controller is _connected_ to the document when both of the following conditions are true:

* its element is present in the document (i.e., a descendant of `document.documentElement`, the `<html>` element)
* its identifier is present in the element's `data-controller` attribute

When a controller becomes connected, Stimulus calls its `connect()` method.

### Targets

A target is _connected_ to the document when both of the following conditions are true:

* its element is present in the document as a descendant of its corresponding controller's element
* its identifier is present in the element's `data-{identifier}-target` attribute

When a target becomes connected, Stimulus calls its controller's `[name]TargetConnected()` method, passing the target element as a parameter. The `[name]TargetConnected()` lifecycle callbacks will fire *before* the controller's `connect()` callback.

## Disconnection

A connected controller will later become _disconnected_ when either of the preceding conditions becomes false, such as under any of the following scenarios:

* the element is explicitly removed from the document with `Node#removeChild()` or `ChildNode#remove()`
* one of the element's parent elements is removed from the document
* one of the element's parent elements has its contents replaced by `Element#innerHTML=`
* the element's `data-controller` attribute is removed or modified
* the document installs a new `<body>` element, such as during a Turbo page change

When a controller becomes disconnected, Stimulus calls its `disconnect()` method.

### Targets

A connected target will later become _disconnected_ when either of the preceding conditions becomes false, such as under any of the following scenarios:

* the element is explicitly removed from the document with `Node#removeChild()` or `ChildNode#remove()`
* one of the element's parent elements is removed from the document
* one of the element's parent elements has its contents replaced by `Element#innerHTML=`
* the element's `data-{identifier}-target` attribute is removed or modified
* the document installs a new `<body>` element, such as during a Turbo page change

When a target becomes disconnected, Stimulus calls its controller's `[name]TargetDisconnected()` method, passing the target element as a parameter. The `[name]TargetDisconnected()` lifecycle callbacks will fire *after* the controller's `disconnect()` callback.

## Reconnection

A disconnected controller may become connected again at a later time.

When this happens, such as after removing the controller's element from the document and then re-attaching it, Stimulus will reuse the element's previous controller instance, calling its `connect()` method multiple times.

Similarly, a disconnected target may be connected again at a later time. Stimulus will invoke its controller's `[name]TargetConnected()` method multiple times.

## Order and Timing

Stimulus watches the page for changes asynchronously using the [DOM `MutationObserver` API](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver).

This means that Stimulus calls your controller's lifecycle methods asynchronously after changes are made to the document, in the next [microtask](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/) following each change.

Lifecycle methods still run in the order they occur, so two calls to a controller's `connect()` method will always be separated by one call to `disconnect()`. Similarly, two calls to a controller's `[name]TargetConnected()` for a given target will always be separated by one call to `[name]TargetDisconnected()` for that same target.
---
permalink: /reference/outlets.html
order: 04
---

# Outlets

_Outlets_ let you reference Stimulus _controller instances_ and their _controller element_ from within another Stimulus Controller by using CSS selectors.

The use of Outlets helps with cross-controller communication and coordination as an alternative to dispatching custom events on controller elements.

They are conceptually similar to [Stimulus Targets](https://stimulus.hotwired.dev/reference/targets) but with the difference that they reference a Stimulus controller instance plus its associated controller element.

<meta data-controller="callout" data-callout-text-value='data-chat-user-status-outlet=".online-user"'>
<meta data-controller="callout" data-callout-text-value='class="online-user"'>


```html
<div>
  <div class="online-user" data-controller="user-status">...</div>
  <div class="online-user" data-controller="user-status">...</div>
  ...
</div>

...

<div data-controller="chat" data-chat-user-status-outlet=".online-user">
  ...
</div>
```

While a **target** is a specifically marked element **within the scope** of its own controller element, an **outlet** can be located **anywhere on the page** and doesn't necessarily have to be within the controller scope.

## Attributes and Names

The `data-chat-user-status-outlet` attribute is called an _outlet attribute_, and its value is a [CSS selector](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors) which you can use to refer to other controller elements which should be available as outlets on the _host controller_. The outlet identifier in the host controller must be the same as the target controller's identifier.

```html
data-[identifier]-[outlet]-outlet="[selector]"
```

<meta data-controller="callout" data-callout-text-value='data-chat-user-status-outlet=".online-user"'>


```html
<div data-controller="chat" data-chat-user-status-outlet=".online-user"></div>
```

## Definitions

Define controller identifiers in your controller class using the `static outlets` array. This array declares which other controller identifiers can be used as outlets on this controller:

<meta data-controller="callout" data-callout-text-value='static outlets'>
<meta data-controller="callout" data-callout-text-value='"user-status"'>
<meta data-controller="callout" data-callout-text-value='userStatus'>


```js
// chat_controller.js

export default class extends Controller {
  static outlets = [ "user-status" ]

  connect () {
    this.userStatusOutlets.forEach(status => ...)
  }
}
```

## Properties

For each outlet defined in the `static outlets` array, Stimulus adds five properties to your controller, where `[name]` corresponds to the outlet's controller identifier:

| Kind | Property name | Return Type | Effect
| ---- | ------------- | ----------- | -----------
| Existential | `has[Name]Outlet` | `Boolean` | Tests for presence of a `[name]` outlet
| Singular | `[name]Outlet` | `Controller` | Returns the `Controller` instance of the first `[name]` outlet or throws an exception if none is present
| Plural | `[name]Outlets` | `Array<Controller>` | Returns the `Controller` instances of all `[name]` outlets
| Singular | `[name]OutletElement` | `Element` | Returns the Controller `Element` of the first `[name]` outlet or throws an exception if none is present
| Plural | `[name]OutletElements` | `Array<Element>` | Returns the Controller `Element`'s of all `[name]` outlets

**Note:** For nested Stimulus controller properties, make sure to omit namespace delimiters in order to correctly access the referenced outlet:

```js
// chat_controller.js

export default class extends Controller {
  static outlets = [ "admin--user-status" ]

  selectAll(event) {
    // returns undefined
    this.admin__UserStatusOutlets

    // returns controller reference
    this.adminUserStatusOutlets
  }
}
```

## Accessing Controllers and Elements

Since you get back a `Controller` instance from the `[name]Outlet` and `[name]Outlets` properties you are also able to access the Values, Classes, Targets and all of the other properties and functions that controller instance defines:

```js
this.userStatusOutlet.idValue
this.userStatusOutlet.imageTarget
this.userStatusOutlet.activeClasses
```

You are also able to invoke any function the outlet controller may define:

```js
// user_status_controller.js

export default class extends Controller {
  markAsSelected(event) {
    // ...
  }
}

// chat_controller.js

export default class extends Controller {
  static outlets = [ "user-status" ]

  selectAll(event) {
    this.userStatusOutlets.forEach(status => status.markAsSelected(event))
  }
}
```

Similarly with the Outlet Element, it allows you to call any function or property on [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element):

```js
this.userStatusOutletElement.dataset.value
this.userStatusOutletElement.getAttribute("id")
this.userStatusOutletElements.map(status => status.hasAttribute("selected"))
```

## Outlet Callbacks

Outlet callbacks are specially named functions called by Stimulus to let you respond to whenever an outlet is added or removed from the page.

To observe outlet changes, define a function named `[name]OutletConnected()` or `[name]OutletDisconnected()`.

```js
// chat_controller.js

export default class extends Controller {
  static outlets = [ "user-status" ]

  userStatusOutletConnected(outlet, element) {
    // ...
  }

  userStatusOutletDisconnected(outlet, element) {
    // ...
  }
}
```

### Outlets are Assumed to be Present

When you access an Outlet property in a Controller, you assert that at least one corresponding Outlet is present. If the declaration is missing and no matching outlet is found Stimulus will throw an exception:

```html
Missing outlet element "user-status" for "chat" controller
```

### Optional outlets

If an Outlet is optional or you want to assert that at least Outlet is present, you must first check the presence of the Outlet using the existential property:

```js
if (this.hasUserStatusOutlet) {
  this.userStatusOutlet.safelyCallSomethingOnTheOutlet()
}
```

### Referencing Non-Controller Elements

Stimulus will throw an exception if you try to declare an element as an outlet which doesn't have a corresponding `data-controller` and identifier on it:

<meta data-controller="callout" data-callout-text-value='data-chat-user-status-outlet="#user-column"'>
<meta data-controller="callout" data-callout-text-value='id="user-column"'>


```html
<div data-controller="chat" data-chat-user-status-outlet="#user-column"></div>

<div id="user-column"></div>
```

Would result in:
```html
Missing "data-controller=user-status" attribute on outlet element for
"chat" controller`
```
---
permalink: /reference/targets.html
order: 03
---

# Targets

_Targets_ let you reference important elements by name.

<meta data-controller="callout" data-callout-text-value="search.query">
<meta data-controller="callout" data-callout-text-value="search.errorMessage">
<meta data-controller="callout" data-callout-text-value="search.results">

```html
<div data-controller="search">
  <input type="text" data-search-target="query">
  <div data-search-target="errorMessage"></div>
  <div data-search-target="results"></div>
</div>
```

## Attributes and Names

The `data-search-target` attribute is called a _target attribute_, and its value is a space-separated list of _target names_ which you can use to refer to the element in the `search` controller.

<meta data-controller="callout" data-callout-text-value="search">
<meta data-controller="callout" data-callout-text-value="results">

```html
<div data-controller="s​earch">
  <div data-search-target="results"></div>
</div>
```

## Definitions

Define target names in your controller class using the `static targets` array:

```js
// controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = [ "query", "errorMessage", "results" ]
  // …
}
```

## Properties

For each target name defined in the `static targets` array, Stimulus adds the following properties to your controller, where `[name]` corresponds to the target's name:

Kind        | Name                   | Value
----------- | ---------------------- | -----
Singular    | `this.[name]Target`    | The first matching target in scope
Plural      | `this.[name]Targets`   | An array of all matching targets in scope
Existential | `this.has[Name]Target` | A boolean indicating whether there is a matching target in scope

<br>**Note:** Accessing the singular target property will throw an error when there is no matching element.

## Shared Targets

Elements can have more than one target attribute, and it's common for targets to be shared by multiple controllers.

<meta data-controller="callout" data-callout-text-value="data-search-target=&quot;projects&quot;">
<meta data-controller="callout" data-callout-text-value="data-search-target=&quot;messages&quot;">
<meta data-controller="callout" data-callout-text-value="data-checkbox-target=&quot;input&quot;">

```html
<form data-controller="search checkbox">
  <input type="checkbox" data-search-target="projects" data-checkbox-target="input">
  <input type="checkbox" data-search-target="messages" data-checkbox-target="input">
  …
</form>
```

In the example above, the checkboxes are accessible inside the `search` controller as `this.projectsTarget` and `this.messagesTarget`, respectively.

Inside the `checkbox` controller, `this.inputTargets` returns an array with both checkboxes.

## Optional Targets

If your controller needs to work with a target which may or may not be present, condition your code based on the value of the existential target property:

```js
if (this.hasResultsTarget) {
  this.resultsTarget.innerHTML = "…"
}
```

## Connected and Disconnected Callbacks

Target _element callbacks_ let you respond whenever a target element is added or
removed within the controller's element.

Define a method `[name]TargetConnected` or `[name]TargetDisconnected` in the controller, where `[name]` is the name of the target you want to observe for additions or removals. The method receives the element as the first argument.

Stimulus invokes each element callback any time its target elements are added or removed. When the controller is connected or disconnected from the document, these callbacks are invoked *before* `connect()` and *after* `disconnect()` lifecycle hooks.

```js
export default class extends Controller {
  static targets = [ "item" ]

  itemTargetConnected(element) {
    this.sortElements(this.itemTargets)
  }

  itemTargetDisconnected(element) {
    this.sortElements(this.itemTargets)
  }

  // Private
  sortElements(itemTargets) { /* ... */ }
}
```

**Note** During the execution of `[name]TargetConnected` and
`[name]TargetDisconnected` callbacks, the `MutationObserver` instances behind
the scenes are paused. This means that if a callback add or removes a target
with a matching name, the corresponding callback _will not_ be invoked again.

## Naming Conventions

Always use camelCase to specify target names, since they map directly to properties on your controller:

```html
<span data-search-target="camelCase"></span>
<span data-search-target="do-not-do-this"></span>
```

```js
export default class extends Controller {
  static targets = [ "camelCase" ]
}
```
---
permalink: /reference/using-typescript.html
order: 07
---

# Using Typescript

Stimulus itself is written in [TypeScript](https://www.typescriptlang.org/) and provides types directly over its package.
The following documentation shows how to define types for Stimulus properties.

## Define Controller Element Type

By default, the `element` of the controller is of type `Element`. You can override the type of the controller element by specifiying it as a [Generic Type](https://www.typescriptlang.org/docs/handbook/2/generics.html). For example, if the element type is expected to be a `HTMLFormElement`:

<meta data-controller="callout" data-callout-text-value="Controller<HTMLFormElement>">

```ts
import { Controller } from "@hotwired/stimulus"

export default class MyController extends Controller<HTMLFormElement> {
  submit() {
    new FormData(this.element)
  }
}
```

## Define Value Properties

You can define the properties of configured values using the TypeScript `declare` keyword. You just need to define the properties if you are making use of them within the controller.

```ts
import { Controller } from "@hotwired/stimulus"

export default class MyController extends Controller {
  static values = {
    code: String
  }

  declare codeValue: string
  declare readonly hasCodeValue: boolean
}
```

> The `declare` keyword avoids overriding the existing Stimulus property, and just defines the type for TypeScript.

## Define Target Properties

You can define the properties of configured targets using the TypeScript `declare` keyword. You just need to define the properties if you are making use of them within the controller.

 The return types of the `[name]Target` and `[name]Targets` properties can be any inheriting from the `Element` type. Choose the best type which fits your needs. Pick either `Element` or `HTMLElement` if you want to define it as a generic HTML element.

```ts
import { Controller } from "@hotwired/stimulus"

export default class MyController extends Controller {
  static targets = [ "input" ]

  declare readonly hasInputTarget: boolean
  declare readonly inputTarget: HTMLInputElement
  declare readonly inputTargets: HTMLInputElement[]
}
```

> The `declare` keyword avoids overriding the existing Stimulus property, and just defines the type for TypeScript.

## Custom properties and methods

Other custom properties can be defined the TypeScript way on the controller class:

<meta data-controller="callout" data-callout-text-value="container: HTMLElement">

```ts
import { Controller } from "@hotwired/stimulus"

export default class MyController extends Controller {
  container: HTMLElement
}
```

Read more in the [TypeScript Documentation](https://www.typescriptlang.org/docs/handbook/intro.html).
---
permalink: /reference/values.html
order: 05
---

# Values

You can read and write [HTML data attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*) on controller elements as typed _values_ using special controller properties.

<meta data-controller="callout" data-callout-text-value="data-loader-url-value=&quot;/messages&quot;">

```html
<div data-controller="loader" data-loader-url-value="/messages">
</div>
```

As per the given HTML snippet, remember to place the data attributes for values on the same element as the `data-controller` attribute.

<meta data-controller="callout" data-callout-text-value="static values = { url: String }">
<meta data-controller="callout" data-callout-text-value="this.urlValue">

```js
// controllers/loader_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = {
    url: String
  }

  connect() {
    fetch(this.urlValue).then(/* … */)
  }
}
```

## Definitions

Define values in a controller using the `static values` object. Put each value's _name_ on the left and its _type_ on the right.

```js
export default class extends Controller {
  static values = {
    url: String,
    interval: Number,
    params: Object
  }

  // …
}
```

## Types

A value's type is one of `Array`, `Boolean`, `Number`, `Object`, or `String`. The type determines how the value is transcoded between JavaScript and HTML.

| Type    | Encoded as…              | Decoded as…                             |
| ------- | ------------------------ | --------------------------------------- |
| Array   | `JSON.stringify(array)`  | `JSON.parse(value)`                     |
| Boolean | `boolean.toString()`     | `!(value == "0" \|\| value == "false")` |
| Number  | `number.toString()`      | `Number(value.replace(/_/g, ""))`       |
| Object  | `JSON.stringify(object)` | `JSON.parse(value)`                     |
| String  | Itself                   | Itself                                  |

## Properties and Attributes

Stimulus automatically generates getter, setter, and existential properties for each value defined in a controller. These properties are linked to data attributes on the controller's element:

Kind | Property name | Effect
---- | ------------- | ------
Getter | `this.[name]Value` | Reads `data-[identifier]-[name]-value`
Setter | `this.[name]Value=` | Writes `data-[identifier]-[name]-value`
Existential | `this.has[Name]Value` | Tests for `data-[identifier]-[name]-value`

### Getters

The getter for a value decodes the associated data attribute into an instance of the value's type.

If the data attribute is missing from the controller's element, the getter returns a _default value_, depending on the value's type:

Type | Default value
---- | -------------
Array | `[]`
Boolean | `false`
Number | `0`
Object | `{}`
String | `""`

### Setters

The setter for a value sets the associated data attribute on the controller's element.

To remove the data attribute from the controller's element, assign `undefined` to the value.

### Existential Properties

The existential property for a value evaluates to `true` when the associated data attribute is present on the controller's element and `false` when it is absent.

## Change Callbacks

Value _change callbacks_ let you respond whenever a value's data attribute changes.

Define a method `[name]ValueChanged` in the controller, where `[name]` is the name of the value you want to observe for changes. The method receives its decoded value as the first argument and the decoded previous value as the second argument.

Stimulus invokes each change callback after the controller is initialized and again any time its associated data attribute changes. This includes changes as a result of assignment to the value's setter.

```js
export default class extends Controller {
  static values = { url: String }

  urlValueChanged() {
    fetch(this.urlValue).then(/* … */)
  }
}
```

### Previous Values

You can access the previous value of a `[name]ValueChanged` callback by defining the callback method with two arguments in your controller.

```js
export default class extends Controller {
  static values = { url: String }

  urlValueChanged(value, previousValue) {
    /* … */
  }
}
```

The two arguments can be named as you like. You could also use `urlValueChanged(current, old)`.

## Default Values

Values that have not been specified on the controller element can be set by defaults specified in the controller definition:

```js
export default class extends Controller {
  static values = {
    url: { type: String, default: '/bill' },
    interval: { type: Number, default: 5 },
    clicked: Boolean
  }
}
```

When a default is used, the expanded form of `{ type, default }` is used. This form can be mixed with the regular form that does not use a default.

## Naming Conventions

Write value names as camelCase in JavaScript and kebab-case in HTML. For example, a value named `contentType` in the `loader` controller will have the associated data attribute `data-loader-content-type-value`.
