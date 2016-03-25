# Ruzuku JavaScript style guide

These are guidelines, not gospel, and can be adjusted/improved over time.
Thoughtful challenges and changes to this document are encouraged.

First, see [AirBnb's JavaScript style guide](https://github.com/airbnb/javascript).

We only support the latest 2 versions of any browser (currently, IE 10+), so
disregard any AirBnb rules that are there as IE9- workarounds.

## Differences

Places where Ruzuku differs from AirBnb are noted below.

### [Needs review] Semicolons

Semicolons are optional. Yes, omitting them does open up a certain class of
bugs, but those bugs are avoided if following the guidelines, and semicolon-less
code reads more easily.

We are using modules, so file concatenation (probably the biggest source of
semicolon bugs) should not be an issue.

The following rule is a corollary.

### Don't use trailing commas

Trailing commas are better for diffs, and make moving code around a bit easier... They still ain't got no alibi... they're ugly.

```js
// Bad
const foo = {
  name: 'Joe',
  age: 'Shmo',
}

// Good
const foo = {
  name: 'Joe',
  age: 'Shmo'
}

```

### Never start a statement with one of these: `([+-`

Never start a new statement with any of these characters: `([+-`. There is
really never a good reason to do this, and there is a very good reason not to.
If you are omitting semicolons, then starting statements with any `([+-` char
could lead to trouble.

```js
// bad/bug
var firstName = 'Chris'
var lastName = 'Davies'
var fullName = firstName + ' ' + lastName
(console.log(fullName))

// good
var firstName = 'Chris'
var fullName = firstName + ' ' + lastName
console.log(fullName)
```

The reason is that without semicolons, this code:

```js
var fullName = firstName + ' ' + lastName
(console.log(fullName))
```

Is understood by JavaScript to mean this:

```js
var fullName = firstName + ' ' + lastName(console.log(fullName))
```

JavaScript things `lastName` is a function being passed a single parameter
(the result of console.log).

### Don't use multiline `/* */` comments

Avoid use of `/* ...  */` style comments. These make commenting / uncommenting
large blocks of code tedious if multiple multi-line comments already exist.

```js
/*
  If you use these, and then try to use the /* to comment out an
  entire file or something, the termination of this comment will
  mess with things...
*/
const noBueno = true

// This is preferred. With any decent editor, pressing return will
// automatically create a new comment line for you
const oneCommentStyle = 'Winning!'
```

### Don't use leading underscores

If it's really, really important that a property is private, capture it in
a closure. Otherwise, give it a `camelCase` name and call it a day.

```js
// Bad
this._hiddenProp = 'Whatevz'

// Better
this.hiddenProp = 'Whatevz'

// Best, captured in a closure
function Foo() {
  let hiddenProp = 'Whatevz'

  return () => console.log(hiddenProp)
}
```

### Prefer pure functions

As much as possible, prefer pure functions to methods. Our models will break
this rule, as we are using Mobx and mutable domain objects, but keep this in
mind as a general rule.

```js
// bad
let prev = 23

function accumulate(next) {
  prev += next
}

// good
function accumulate(prev, next) {
  return prev + next
}

```

## [Needs review] Classes

The following are general rules of thumb for creating and using classes.

### Constructors should be functions

Constructors should be functions (e.g. can be passed to `map` and friends)

```js
// bad, cannot do something like ['Chris', 'Kamal'].map(User)
function User(name) {
  this.name = name
}

// good, can do ['Chris', 'Kamal'].map(User)
function User(name) {
  return {
    name
  }
}
```

### Methods should be functions

As much as possible, methods should be written in such a way that they can be
called as a function without requiring binding.

```js
// bad, cannot do something like onClick={user.toggleIsActive}
function User(isActive) {
  this.isActive = isActive
}

User.prototype = {
  toggleIsActive () {
    this.isActive = !this.isActive
  }
}

let toggle = new User(true).toggleIsActive

toggle() // produces an error

// good, toggleIsActive can be passed without worrying about binding
function User(isActive) {
  const me = {
    isActive,

    toggleIsActive () {
      me.isActive = !me.isActive
    }
  }

  return me
}

let toggle = User(true).toggleIsActive

toggle() // toggles correctly

```

### Avoid the use of `this`

As much as possible, avoid the use of `this` keyword. This means using closures
to define objects. This does incur a memory and slight performance penalty, but
it provides the following benefits:

- Constructors are just functions and can be directly passed (e.g. to `map`)
- Methods are just functions and can be directly passed (e.g. to `onClick`)
- No need to worry about binding

```js
// bad, cannot do something like ['Chris', 'Kamal'].map(User)
function User(name) {
  this.name = name
}

// good, can do ['Chris', 'Kamal'].map(User)
function User(name) {
  return {
    name
  }
}
```

### Avoid using ES6 classes

Unless there is a compelling reason, avoid using ES6 classes, as they pretty
much require the use of `this` which is difficult to reason about in JS,
especially when switching between languages with better class support
(Ruby, C#, etc).

### Explicitly define fields, unless using an object as a generic hash/map

When programming, explicit is generally better than implicit for maintenance.
Explicitly defining an object's fields makes the object definition much clearer.
It also is a step towards static typing via TypeScript/Flow which we may
incorporate in the future. 

```js
// bad... who knows what shape a User object might be?
function User(spec) {
  const me = {
    ...spec,
    get fullName () {
      return `${me.firstName} ${me.lastName}`
    }
  }

  return me
}

// good... more typing up front, but much more understandable
function User(spec) {
  const me = {
    id: spec.id,
    firstName: spec.firstName,
    lastName: spec.lastName,
    age: spec.age,

    get fullName () {
      return `${me.firstName} ${me.lastName}`
    },
  }

  return me
}

// best... eliminates two potential sources of typos (typing spec. and
// referring to an undefined property (e.g. spec.firsName))
function User({id, firstName, lastName, age}) {
  const me = {
    id,
    firstName,
    lastName,
    age,

    get fullName () {
      return `${me.firstName} ${me.lastName}`
    },
  }

  return me
}

```

### Multi-file components should have their own folder

When writing a complex, multi-file component, it should live in its own folder
and expose its external interface via `index.js` or `index.jsx`. This file is
the `main` entry point for the component. Outside scripts should never refer
to anything other than that which is exposed from `index`.

```js
// bad
import {Foo} from '../some-component/foo'

// good (this is shorthand for '../some-component/index')
import {foo} from '../some-component'
```

## JSX

When writing React/JSX code, here are some rules of thumb.

### Wrap return statements in parenthesis

```js
// bad
export default function ({text}) {
  return <div className="foo">
    {text}
  </div>
}

// good
export default function ({text}) {
  return (
    <div className="foo">
      {text}
    </div>
  )
}
```

### Do not use ternary condition to toggle between two or more elements

Do not use ternary condition to choose which of two or more elements to show:

```js
// bad
function Foo({isSelected, select}) {
  return (
    <span>
      {isSelected ?
        <span>Selected</span> :
        <button onClick={select}>Select me!</button>
      }
    </span>
  )
}

// good
function Foo({isSelected, select}) {
  if (isSelected) {
    return (
      <span>Selected</span>
    )
  }

  return (
    <button onClick={select}>Select me!</button>
  )
}
```

### [Needs review] Only use ternary when optionally showing a single element

?Should we simply disallow ternary conditions in JSX?

Only use ternary conditions when you want to optionally show an element, and
in this case, always put the `undefined` clause first:

```js
// bad
function Bar({isSelected}) {
  return (
    <div>
      Some item
      {isSelected ?
        <span className="status-badge">
          Selected
        </span>
        : undefined
      }
    </div>
  )
}

// good
function Bar({isSelected}) {
  return (
    <div>
      Some item
      {renderSelectedBadge(isSelected)}
    </div>
  )
}

function renderSelectedBadge(isSelected) {
  if (isSelected) {
    return (
      <span className="status-badge">
        Selected
      </span>
    )
  }
}


// good
function Bar({isSelected}) {
  return (
    <div>
      Some item
      {!isSelected ? undefined :
        <span className="status-badge">
          Selected
        </span>
      }
    </div>
  )
}

```

### Use React state sparingly

If you're using `this.state` it's sign of code-smell. But there are valid uses
such as when writing a ver specific component. An example is the
`ConfirmableButton` component, which when clicked shows a confirmation dialog
and only triggers its `onConfirm` event when the dialog has been confirmed.

In such a component it's perfectly acceptable to track dialog state
via `this.state`.

### Prefer functions over components

You can write React components using a single render function,
`React.createClass`, and ES6 class syntax. Functions are preferable.

```js
// this should be a function
export default React.createClass({
  render: () {
    return (
      <button className={'btn ' + this.props.className}>
        {this.props.children}
      </button>
    )
  }
})

// better
export default function ({className, children})
  return (
    <button className={'btn ' + className}>
      {children}
    </button>
  )
})

```

### React class-based components

When creating React components as classes, it is OK to use the `this` keyword
and full-fledged classes. This is in exception to our general rule about
preferring closures. The choice is yours.

There are a few reasons why you may want to make this exception: React creates
these classes on each render, so efficiency matters here. Also, React components
aren't passed around or used outside of their own file, except as JSX,  so
convenient constructor functions and methods-as-functions matter less.
