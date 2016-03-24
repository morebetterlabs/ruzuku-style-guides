# Ruzuku CSS/SCSS style guide

First, see [AirBnb's CSS/SCSS style guide](https://github.com/airbnb/css).

## General

### As much as possible, write pure CSS

SCSS is powerful and easily abused:

- Don't nest selectors
- Don't perform complex logic/computation
- Don't get clever with abstractions
- *Do* prefer SCSS' `//` comments to `/* */`

If a bit of SCSS feels clever, it's very possibly too clever, so you may want
to get a second opinion.

## Be careful when using mixins

In CSS, similarities are often coincidental, and overuse of mixins means that
making a change to a mixin often has unwanted consequences due to it used to
DRY up some rules that really were unrelated to the mixin's intent.

### As much as possible, keep CSS isolated

- General rules/defaults should rarely be written or modified.
- Use a naming convention to isolate your styles

An example for a `user-widget`:

```scss
// bad
.user-widget {
  border: 1px solid #CCC;
  padding: 1em;
}

  .name {
    display: block;
    opacity: 0.5;
  }

  .pic {
    width: 2em;
    height: 2em;
  }

// Good
.user-widget {
  border: 1px solid #CCC;
  padding: 1em;
}

  .user-widget-name {
    display: block;
    opacity: 0.5;
  }

  .user-widget-pic {
    width: 2em;
    height: 2em;
  }

// Also fine to use BEM:
.user-widget {
  border: 1px solid #CCC;
  padding: 1em;
}

  .user-widget__name {
    display: block;
    opacity: 0.5;
  }

  .user-widget__pic {
    width: 2em;
    height: 2em;
  }

```

### Use SCSS' `&` feature sparingly

We want our SCSS to be as searchable as possible. If I see class `user-name` in
our HTML, I should be able to search for `.user-name` and find that rule.

```scss
// Bad
.user {
  border: 1px solid #DDD;

  // This generates .user-name, but is impossible to find via Cmd+F
  &-name {
    opacity: 0.5;
  }
}

// Good
.user {
  border: 1px solid #DDD;
}

  .user-name {
    opacity: 0.5;
  }

```

Usage of `&` is encouraged for elements:

```scss
// Perfectly fine
.profile-link {
  color: blue;

  &:hover {
    color: orange;
  }
}
```

It is also acceptable when writing modifier classes such as `.user.is-current`,
as searchability is not impaired:

```scss
// Fine
.user {
  color: #333;

  &.is-current {
    text-decoration: underline;
  }
}
```

### Don't use vendor prefixes

We are using [autoprefixer](https://github.com/postcss/autoprefixer).

```scss
// Bad
-ms-opacity: 0.5;
opacity: 0.5;

// Good
opacity: 0.5;
```

### Unless defining defaults, don't select by tag name

```scss
// bad
.login-form input {
  border: 2px solid blue;
}

// good
.login-form__textbox {
  border: 2px solid blue;
}
```

### Do not put units after a zero

If you have a `0`, never follow it with units.

```scss
// Bad
border: 0px;

// Good
border: 0;
```

### If you have a decimal, use a leading zero

```scss
// Bad
font-size: .9em;

// Good
font-size: 0.9em;
```

## Further reading

- http://cssguidelin.es/
- http://sass-guidelin.es/
