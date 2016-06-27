---
title: Consumer-Friendly API's in Sass - Runtime Contracts for Functions & Mixins
date: 2016-06-25
author: Scotty Eckenthal
email: scott.eckenthal@gmail.com
template: article
issue: 0
---

This is the first in a series of articles written to discuss practices for delivering consumer-friendly API's in Sass libraries and modules. Often in the Front-End world, we don't think of API design as something that applies to our CSS or, for that matter, our Sass. This series will seek to debunk that notion, discussing a few basic points that, when implemented, can greatly increase the usability of your module.

To complement this series, I've built [**scss-grid-gen**](https://github.com/scottyeck/scss-grid-gen). It is a node-module that, once installed and imported, exposes a dead-simple Sass API that allows a user hopefully easy access to customizable grids. It is, I know, just another grid framework. I do not expect it to get much use, which is fine. I've built it solely for the example of demonstrating practices discussed.

With each installment of the series that is released, I will update the module and re-release with the improvements as discussed.

## Part 1: Runtime Contracts for Functions & Mixins

When distributing a tool, one of the best things we can do for our users is to provide them fast and descriptive feedback when they are using a tool incorrectly. For the consumer, this simply makes their life easier. When confronted with an error, a user is far more likely to be able to resolve it quickly and easily if the corresponding message is descriptive and perhaps even instructive. For the maintainer, this might lessen the number of unncessarily submitted bugs or support requests.

This, like anything else, can apply to our Sass. One of the simplest ways of doing so is type-checking the arguments passed to a function (or in Sass, a mixin as well) at runtime. This practice is not new - in fact, it is all but an expected convention in most contexts.

In this article, we will discuss the ways in which these practices are employed in _scss-grid-gen_. For the impatient, the changes outlined here can be easily seen in the upgrade between version [_0.0.2_](https://github.com/scottyeck/scss-grid-gen/tree/v0.0.2) and [_0.1.0_](https://github.com/scottyeck/scss-grid-gen/tree/v0.1.0).

### Context

Consider this mixin imported from _scss-grid-gen_.

```scss
// scss-grid-gen/_core.scss

// 
// Applies properties to an individual grid-column element
// based on the number of columns we wish for it to span.
// 
// @param $column-span	(Number)	The desired span count
// @param $column-count	(Number)	The total of columns
// 
@mixin grid-gen-column-width(
	$column-span: 	null,
	$column-count: 	$grid-gen-column-count) {

	width: 100% / $column-count * $column-span;
}
```

This mixin simply takes the number of columns a particular grid-element is supposed to span and converts it into a percentage based on the total number of columns available in the grid. It then assigns that value to a CSS `width` property so it may be used. Simple enough, right?

### The problem...

Below, we see an example of how this mixin might be used. (For the purpose of simplicity, we'll be demonstrating with a relatively useless 4-column grid.)

```scss
// app.scss

$grid-gen-column-count: 4;

@import "node_modules/scss-grid-gen/core";

.col-1 { @include grid-gen-column-width(); }
.col-2 { @include grid-gen-column-width(); }
.col-3 { @include grid-gen-column-width(); }
.col-4 { @include grid-gen-column-width(); }
```

At compile time, this errors out. This is good! The user hasn't specified the number of columns each invocation desires. However, the error message isn't all that helpful...

```bash
{
  "formatted": "Error: Invalid null operation: \"25% times null\".\n        on line 115 of node_modules/scss-grid-gen/src/_grid-gen.scss\n>> \twidth: 100% / $column-count * $column-span;\n   --------^\n",
  "message": "Invalid null operation: \"25% times null\".",
  "column": 9,
  "line": 115,
  "file": "/Users/scottyeck/gar/grid-test/node_modules/scss-grid-gen/src/_grid-gen.scss",
  "status": 1
}
```

The error message, which reads `"Invalid null operation: \"25% times null\"."`, tells us that _something_ has not been set up correctly, but it does not give any further indication as to what the problem actually is. Were the consumer to be experienced with Sass, they could dig through the source code and figure out that the mixin was invoked with the default value of `null` rather than a number. This, however, is not something that we want to assume.

How might we provide better feedback to the user as to their error?

### The solution...

To provide the user this feedback, we should check that the user has invoked the mixin correctly at compile-time and throw a helpful error if it doesn't. To do so, we make good use of Sass' built-in `type-of` function, as well as the `@error` directive, which will halt the compilation process.

```scss
// scss-grid-gen/_core.scss

// 
// Applies properties to an individual grid-column element
// based on the number of columns we wish for it to span.
// 
// @param $column-span	(Number)	The desired span count
// @param $column-count	(Number)	The total of columns
// 
@mixin grid-gen-column-width(
	$column-span: 	null,
	$column-count: 	$grid-gen-column-count) {

	@if (type-of($column-span) != number) {
		@error "Mixin `grid-gen-column-width` expected arg `$column-span` to be of type `number` but received `" + $column-span + "`."
	}

	@if (type-of($column-count) != number) {
		@error "Mixin `grid-gen-column-width` expected arg `$column-count` to be of type `number` but received `" + $column-span + "`."
	}

	@if (not ($column-span <= $column-count)) {
		@error "Mixin `grid-gen-column-width` expected arg `$column-span` to be less than or equal to arg `$column-count`."
	}

	width: 100% / $column-count * $column-span;
}
```

Here, we've modified our mixin to check the types of both `$column-span` and `$column-count`, and we've also checked to make sure that `$column-span` is in fact within the range allowed by `$column-count` to eliminate the possibility of nasty semantic errors.

If our Sass from before is not changed, then the code, again, should error out. This time, however, we should receive a far more descriptive message.

```bash
{
  "formatted": "Error: Invalid null operation: \"\"Mixin `grid-gen-column-width` expected arg `$column-span` to be of type `number` but received `\" plus null\".\n        on line 116 of node_modules/scss-grid-gen/src/_grid-gen.scss\n>> \t\t@error \"Mixin `grid-gen-column-width` expected arg `$column-span` to be of t\n   ---------^\n",
  "message": "Invalid null operation: \"\"Mixin `grid-gen-column-width` expected arg `$column-span` to be of type `number` but received `\" plus null\".",
  "column": 10,
  "line": 116,
  "file": "/Users/scottyeck/gar/grid-test/node_modules/scss-grid-gen/src/_grid-gen.scss",
  "status": 1
}
```

The user now sees that their invocation of the mixin was incorrect, specifically pertaining to their lacking an assignment of the `$column-span` argument. They now know that they should provide the `$column-span` argument as a number. Assuming they're able to work it out, they may re-write their implementation as follows:

```scss
// app.scss

$grid-gen-column-count: 4;

@import "node_modules/scss-grid-gen/core";

.col-1 { @include grid-gen-column-width($column-span: 1); }
.col-2 { @include grid-gen-column-width($column-span: 2); }
.col-3 { @include grid-gen-column-width($column-span: 3); }
.col-4 { @include grid-gen-column-width($column-span: 4); }
```

Our Sass has compiled, and our user has configured pieces of their grid with very few "lines-of-code". The objective has been met!

### Other use cases

There are several examples of improvements made to **scss-grid-gen** that are of a similar vein:

#### Ensuring validity of CSS property values

Consider the following mixin, used to set base properties on a _column_ element inside our grid.

```scss
// app.scss

// 
// Applies base properties to a grid-column-element, primarily
// pertaining to float and exterior padding for gutters.
// 
// @param $flow-direction	(String)	The direction to float
// @param $gutter-width		(Number)	The desired gutter width
// 
@mixin grid-gen-column(
	$flow-direction: 	$grid-gen-flow-direction,
	$gutter-width: 		$grid-gen-gutter-width) {

	box-sizing: border-box;
	display: block;
	float: $flow-direction;
	padding-left: $gutter-width * 0.5;
	padding-right: $gutter-width * 0.5;
}
```

This mixin's job is simple enough. One easy improvement we could make is to ensure that `$gutter-width` is a `number`, as discussed above.

Checking that `$flow-direction` is a string, however, may not be strict enough. The user could, should they wish to stretch the bounds of the tool, supply any value. A `float` value of `jeff` isn't going to be very helpful within the context of our grid.

Luckily, remedying this is easy.

```scss
// app.scss

// 
// Applies base properties to a grid-column-element, primarily
// pertaining to float and exterior padding for gutters.
// 
// @param $flow-direction	(String)	The direction to float
// @param $gutter-width		(Number)	The desired gutter width
// 
@mixin grid-gen-column(
	$flow-direction: 	$grid-gen-flow-direction,
	$gutter-width: 		$grid-gen-gutter-width) {

	$_valid-flow-directions: ("left", "right");

	@if (type-of($gutter-width) != number) {
		@error "Mixin `grid-gen-column-width` expected arg `$gutter-width` to be of type `number` but received `#{$gutter-width}`.";
	}

	@if (not index($_valid-flow-directions, $flow-direction)) {
		@error "Mixin `grid-gen-column-width` expected arg `$flow-direction` to be member of `#{$+_valid-flow-directions}` but received `#{$flow-direction}`.";
	}

	box-sizing: border-box;
	display: block;
	float: $flow-direction;
	padding-left: $gutter-width * 0.5;
	padding-right: $gutter-width * 0.5;
}
```

Here, we've checked to ensure that the value of `$flow-direction`, which will be set as the value of a CSS `float` property, is, in fact, a valid CSS `float` property. We do so by containing the possible values - `"left"` and `"right"` - in a list and ensuring that the value of `$flow-direction` is, in fact, in that list. This check is easy enough, and will result in greater specificity in the API contract as desired.

#### Ensuring existence of Sass function callbacks

One last time, we consider a mixin from **scss-grid-gen**, this time one that generates a class for each column-span option in our grid.

```scss
// 
// Generates a column-span class for each span-count
// possibility based on the total number of columns.
// 
// @param $column-count		(Number)		The desired span count
// @param $class-formatter	(Function name)	The function to generate the
// 											desired class based on span
// 
@mixin grid-gen-column-classes(
	$column-count: 		$grid-gen-column-count,
	$class-formatter: 	_grid-gen-column-class) {

	@for $i from 1 through $column-count {
		$class: call($class-formatter, $i);
		$class: grid-gen-ensure-class($class);

		#{$class} {
			@include grid-gen-column-width(
				$column-count: $column-count,
				$column-span: $i
			);
		}
	}
}
```

As before, we can easily ensure that `$column-count` is a `number`, but dealing with `$class-formatter` is going to be a little trickier. This is because `$class-formatter` is meant to supply the name of a function that the user should provide to format the classnames that the mixin generates.

Below, we demonstrate the necessary improvement...

```scss
// 
// Generates a column-span class for each span-count
// possibility based on the total number of columns.
// 
// @param $column-count		(Number)		The desired span count
// @param $class-formatter	(Function name)	The function to generate the
// 											desired class based on span
// 
@mixin grid-gen-column-classes(
	$column-count: 		$grid-gen-column-count,
	$class-formatter: 	_grid-gen-column-class) {

	@if (type-of($column-count) != number) {
		@error "Mixin `grid-gen-column-classes` expected arg `$column-count` to be of type `number` but received `#{$column-count}`.";
	}

	@if ((type-of($class-formatter) != string) or (not function-exists($class-formatter))) {
		@error "Mixin `grid-gen-column-classes` expected arg `$class-formatter` to be name of user-defined function but received `#{$class-formatter}`.";
	}

	@for $i from 1 through $column-count {
		$class: call($class-formatter, $i);
		$class: grid-gen-ensure-class($class);

		#{$class} {
			@include grid-gen-column-width(
				$column-count: $column-count,
				$column-span: $i
			);
		}
	}
}
```

Not only do we need to check that `$class-formatter` is a `string`, but also that the corresponding `function` exists. This can be accomplished in one unfortunately long line of Sass, but it's easy enough!

### Conclusion

When building a library, taking a little bit of extra care to validate the arguments that are passed to your mixins and functions. It will make the user's life a lot easier! To see this in action in _scss-grid-gen_, take a look at the differences between versions [_0.0.2_](https://github.com/scottyeck/scss-grid-gen/tree/v0.0.2) and [_0.1.0_](https://github.com/scottyeck/scss-grid-gen/tree/v0.1.0) on GitHub.

---

Look out for further entires in this series. In _Part 2_, we will discuss strategies for gracefully deprecating features of your library, and in _Part 3_, we will delve into the somewhat-uncharted waters of unit-testing your Sass.
