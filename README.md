# Inheriting or isolating scope

## Overview

So far, the directives we've made aren't really that useful. We're not doing anything special, such as displaying dynamic data - let's change that!

## Objectives

- Describe scope inheritance
- Describe scope isolate
- Create an inherited scope
- Create an isolate scope
- Bind data to the scope
- Access inherited and isolate scope properties

## Scope inheritance

Now, one cool thing about our directives is that they can display dynamic data. For instance, we might want to have a Twitter card to display a users Twitter handle, along with a link to follow said user.

We can do this exactly like we've done in our views so far - using `{{}}`.

Note: now that we've started using more HTML in our directives, instead of just passing a string, we pass in a multi-line array, joined by an empty string. This means we can have a multi-lined template without having to worry about string concatenation. This is great for small templates, but as our templates get larger we should move out into a separate file.

Lets create the Twitter card.

```js
function TwitterCard() {
	return {
		template: [
			'<div class="twitter">',
				'<a href="https://twitter.com/">Follow @username on twitter!</a>',
			'</div>'
		].join(''),
		restrict: 'E'
	};
}

angular
	.module('app')
	.directive('twitterCard', TwitterCard);
```

Now that's cool, but how do we *actually* get our data? Well, we can actually inherit it from the scope above.

Let's assume we've got a controller, that has a property on it's scope named `twitter`. We would display this in our view as follows:

```html
<div ng-controller="SomeController">
	{{ twitter }}
</div>
```

If we were to use our Twitter card directive instead of displaying `{{ twitter }}`, we could actually use `{{ twitter }}` in our directives template.

```html
<div ng-controller="SomeController">
	<twitter-card></twitter-card>
</div>
```

```js
function TwitterCard() {
	return {
		template: [
			'<div class="twitter">',
				'<a href="https://twitter.com/{{ twitter }}">Follow @{{ twitter }} on Twitter!</a>',
			'</div>'
		].join(''),
		restrict: 'E'
	};
}

angular
	.module('app')
	.directive('twitterCard', TwitterCard);
```

This will result in this HTML being put into the DOM (we're assuming `$scope.twitter` equals `billgates`.)

```html
<div ng-controller="SomeController">
	<twitter-card>
		<div class="twitter">
			<a href="https://twitter.com/billgates">Follow @billgates on Twitter!</a>
		</div>
	</twitter-card>
</div>
```

This is awesome, but what if we want to use our Twitter card twice? Isn't that the point of directives, the ability to reuse them again and again?

One ridiculous way of doing this is to create a new controller for every Twitter card we want to show, assigning all of the different usernames we want to display to their own controller - surely there must be a better way?

## Isolate Scope

Aha! There is! It's called isolate scope. What this does is creates a new scope for our directive - it can either be a completely brand new one or copied from our parent.

In our directive's object, we can attach a property named `scope` to it. This can have a few different values:

### scope: false

If `scope` is equal to `false`, then no new scope is created. This is the same as not having `scope` set (like in our examples above).

### scope: true

If `scope` is equal to `true`, we create a new scope for our directive. This is copied over from the parent scope. If we put `scope` as `true` in our examples above, nothing would change *visually*. However, if we changed the twitter handle, it would update in the controller's scope, but not the directive's.

If we imagine this:

```html
<div ng-controller="SomeController">
	{{ twitter }}
	<twitter-card></twitter-card>
</div>
```

We'd end up with this being rendered:

```html
<div ng-controller="SomeController">
	billgates
	<twitter-card>
		<div class="twitter">
			<a href="https://twitter.com/billgates">Follow @billgates on Twitter!</a>
		</div>
	</twitter-card>
</div>
```

If we were to then update `$scope.twitter` in `SomeController`, we'd end up with:

```html
<div ng-controller="SomeController">
	new value!
	<twitter-card>
		<div class="twitter">
			<a href="https://twitter.com/billgates">Follow @billgates on Twitter!</a>
		</div>
	</twitter-card>
</div>
```

The Twitter handle inside `<twitter-card>` hasn't changed. This is because we've created a new scope (initially based off of the parent scope) - they are no longer linked!

### scope: {}

Now, this is where things get funky. We can pass through an object to our scope object - but why?

When we use normal HTML elements, such as `<input />`, we configure it via attributes. For instance, we might give it a name - we'd do this like such:

```html
<input name="ourInputName" />
```

Hold on a minute.. Can we configure our directives by passing through attributes?

Yes we can!

To do this, we need to specify *what* attributes we want to be configured by. Using our Twitter card above, the only thing we need to configure it is the Twitter handle.

To grab this value out of the attributes, we can specify the attribute name in the object. For instance, if we wanted to utilise the Twitter card as such:

```html
<twitter-card handle="billgates"></twitter-card>
<twitter-card handle="bob"></twitter-card>
```

We'd put the property `handle` into our scope object.

Now, what do we put as the value? There are two that we can use:

#### @ (at)

If we pass through `@` as the value, as so:

```js
function TwitterCard() {
	return {
		template: [
			'<div class="twitter">',
				'<a href="https://twitter.com/{{ handle }}">Follow @{{ handle }} on Twitter!</a>',
			'</div>'
		].join(''),
		scope: {
			handle: '@'
		},
		restrict: 'E'
	};
}

angular
	.module('app')
	.directive('twitterCard', TwitterCard);
```

What `@` tells Angular is to just copy the value as it is. This will literally copy whatever we put in the `handle` attribute and put it into our scope.

#### = (equals)

However, if we want to be able to receive variables through, we can use `=`.

```js
function TwitterCard() {
	return {
		template: [
			'<div class="twitter">',
				'<a href="https://twitter.com/{{ handle }}">Follow @{{ handle }} on Twitter!</a>',
			'</div>'
		].join(''),
		scope: {
			handle: '='
		},
		restrict: 'E'
	};
}

angular
	.module('app')
	.directive('twitterCard', TwitterCard);
```

What this means is that if we have a controller with two properties on it, such as `twitterHandle1` and `twitterHandle2`, we can do this:

```html
<twitter-card handle="twitterHandle1"></twitter-card>
<twitter-card handle="twitterHandle2"></twitter-card>
```

Angular will then change the values over to their actual scope values before passing the value through to our directive. This also means that if we were to update either of them, Angular will automatically update our directive to match it too. Awesome! There's not really any reason why you wouldn't want the directive to update with the latest values, but if you ever do, the option is baked in.

#### Changing attribute names

Notice how we've updated our template to use `{{ handle }}`? This is because our `$scope` has the property `handle` on it now, because of the object.

However, you might not like this - you might have wanted to stick with `{{ twitter }}` but *also* have it configured by using the attribute `handle`. To do this, we can change the object property to `twitter` and we put `handle` after the `@` or `=`, like follows:

```js
function TwitterCard() {
	return {
		template: [
			'<div class="twitter">',
				'<a href="https://twitter.com/{{ twitter }}">Follow @{{ twitter }} on Twitter!</a>',
			'</div>'
		].join(''),
		scope: {
            twitter: '@handle'
        },
		restrict: 'E'
	};
}

angular
	.module('app')
	.directive('twitterCard', TwitterCard);
```

This means we can do this still:

```html
<twitter-card handle="billgates"></twitter-card>
```

Whilst populating `$scope.twitter` with the value. There's no advantage to this or real reason as to why you'd need to use it, but the flexibility is there if you need it.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/angular-inheriting-or-isolate-scope-readme'>Inheriting Or Isolating Scope </a> on Learn.co and start learning to code for free.</p>
