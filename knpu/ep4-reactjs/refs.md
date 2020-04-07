# Refs

Right now, we're using the `name` attribute of each form field to get the underlying
DOM element. We use *that* to fetch its value.

This is interesting: *most* of the time in React, you communicate down to your
elements - and your child components - via props. You use props when you render the
element objects, and React handles creating the *real* DOM elements from that. The
DOM is *not* something we normally touch directly.

But occasionally, you *will* want to access the underlying DOM elements. For example,
you might want to read a value from a form field, call `focus()` on an element, trigger
media playback if you're rendering a video tag or integrate with a third-party
JavaScript library that needs you to pass it a DOM element.

## Creating the refs

Because these are *totally* valid use-cases, React gives us a *great* system to
access any DOM element called `refs`. We need to access two elements: the `select`
element and the `input`. Cool! In the constructor, create 2 new properties:
`this.quantityInput = React.createRef()` and `this.itemSelect = React.createRef()`.

[[[ code('681b6d5bf9') ]]]

This just, "initialized" these two properties. The real magic is next: on the select,
replace the `name` attribute with `ref={this.itemSelect}`. Do the same thing on the
input: move the props onto their own lines, then add `ref={this.quantityInput}`.

[[[ code('9bcd4b592b') ]]]

To really *get* what this does, you need to see it. Comment out the `onNewItemSubmit()`
call for a minute: it's temporarily broken. Then, let's `console.log(this.quantityInput)`
and also `this.itemSelect`.

[[[ code('08f6a1aa5b') ]]]

Moment of truth! Move over, Encore already refreshed the page. Fill out the fields,
hit enter... cool! Each "ref" is an *object* with one property called `current`
that is *set* to the underlying DOM element! Yea, I know, the fact that it sets
the DOM element to a `current` key is a little weird... but it's just how it works.

## Using the Refs

Thanks to this, let's set the DOM element objects onto two new variables:
`const quantityInput = this.quantityInput.current` and
`const itemSelect = this.itemSelect.current`. Below, log the field values:
`quantityInput.value` and, this is a bit harder,
`itemSelect.options[itemSelect.selectedIndex]`, then, `.value`.

[[[ code('3cf6e2e566') ]]]

This finds *which* option is selected, then returns its `value` attribute. Try
it: refresh, select "Big Fat Cat", enter 50 and... boom! People, this is *huge*!
We can *finally* pass *real* information to the callback. Uncomment `onNewItemSubmit`.
Pass the options code, but, change to `.text`: this is the *display* value of the
option. And, until we *actually* starting saving things via AJAX, *that* is what
we'll pass to the callback. Next, use `quantityInput.value`.

[[[ code('627f144b84') ]]]

## Updating the repLogs State

*Finally*, go back to `RepLogApp` and find `handleNewItemSubmit`: *this* is the
function we just called. I'm going to change the argument to `itemLabel`, then
clear things out. Ok, our job here is simple: add the new rep log to the
`repLogs` state. Read the existing state with `const repLogs = this.state.repLogs`.
Then, create the `newRep` set to an object. This needs the same properties as the
other rep logs. So, add `id` set to... hmm. We don't have an id yet! Set this to
"TODO". Then, `reps: reps`, `itemLabel: itemLabel` and, the last field is
`totalWeightLifted`. Hmm, this is another tricky one! Our React app doesn't know
how "heavy" each item is... and so we don't know the `totalWeightLifted`! Later,
we'll need to ask the server for this info. For now, just use a random number.

[[[ code('41145b96e4') ]]]

And finally, let's update the state! `repLogs.push(newRep)` and `this.setState()`
with `repLogs` set to `repLogs`.

[[[ code('3710b546cf') ]]]

Um... there *is* a teeny problem with *how* we're updating the state here. But,
we'll talk about it next. For now, gleefully forget I said *anything* was wrong
and refresh! Fill out the form and... boo! A familiar error:

> Cannot read property state of undefined in RepLogApp line 26

I've been *lazy*. *Each* time we create a handler function in a class, we need to
*bind* it to this! In the constructor, add `this.handleNewItemSubmit =` the same
thing `.bind(this)`.

[[[ code('2ad5551e57') ]]]

## Using uuids

Try it again! We got it! It updates the state and *that* causes React to re-render
and add the row. But... if we try it a second time, it *does* update the state,
but, ah! It yells at us:

> Encountered two children with the same key

Ah! The `id` property is eventually used in `RepLogList` as the `key` prop. And
with the hardcoded `TODO`, it's not unique. Time to fix that temporary hack.

But, hmmm. How *can* we get a unique id? There are always two options. First, you
can make an AJAX request and wait for the server to send back the new id before
updating the state. We'll do that later. *Or*, you can generate a `uuid` in JavaScript.
Let's do that now. And later, when we start talking to the server via AJAX, we'll
discuss how UUIDs *can* still be used, and are a great idea!

To generate a UUID, find your terminal and install a library:

```terminal
yarn add uuid --dev
```

***TIP
In the latest version of `uuid`, you should import the `uuid` package like this:
```javascript
import { v4 as uuid } from 'uuid';
```
***

Wait for that to finish... then go to `RepLogApp` and import `uuid` from
`uuid/v4`. There are a few versions of UUID that behave slightly differently.
It turns out, we want v4.

Down in `constructor()`, use UUID's everywhere, even in our dummy data. Then, move
to the handle function and use it there.

[[[ code('5493624b6d') ]]]

Let's see if this fixes things! Move over, make sure the page is refreshed and
start adding data. Cool: we can add *as* many as we want.

## Clearing the Form

Which... is actually kinda weird: when the form submits, we need the fields to
reset. No problem: in `RepLogCreator`, in addition to *reading* the values off
of the DOM elements, we can also *set* them. At the bottom, use
`quantityInput.value = ''` and `itemSelect.selectedIndex = 0`.

[[[ code('b80e54b86a') ]]]

Try it! Refresh... fill in the form and... sweet! Whenever you need to work
directly with DOM elements, refs are your friend.
