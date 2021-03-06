# Vue.js Notes

## Installation & getting setup

#### Via npm

```bash
$ sudo npm install -g vue
```

#### Via CDN

```html
<!-- development version -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.js"></script>

<!-- production version -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.min.js"></script>
```

#### Vue CLI

The Vue CLI will help you bootstrap a SPA app with Vue:

```bash
$ sudo npm install -g vue-cli
```

Create the app with:

```bash
$ vue init webpack my-project
```

#### Dev tools Chrome extension

Also, grab the [**Vue devtools** Chrome extension](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd).

## Hello world

Vue.js is meant to be used as either just a view layer, the vue core library, or as a full SPA solution by including additional vue libraries.  

The basic hello world in vue uses just the core library, and it looks like this:

```html
<html> 
	<head>
		<title>Hello Vue</title> 
	</head>
	<body>
		<div id="app">
			<h1>{{ message }}</h1>
			<input v-model="message">
		</div>
	</body> 
	<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.min.js"></script>
	<script>
		var data = { message: 'Hello, world!' };
		new Vue({ el: '#app', data: data });
	</script>
</html>
```

## Template directives

#### `v-model`

Places the contents of the data item in the element it is attached to.  There is two way binding on this, so if you update the value on the element, the data value of the Vue object will also get updated.

You can see an example of this in the hello world example above.

#### `v-show`

For conditionally showing or hiding an element.  This directive will apply a CSS style to either show or hide the element:

```html
<div id="app">
	<h1 v-show="!name">What's your name?</h1>
	<h1 v-show="name">Hi, {{ name }}!</h1>
	<input v-model="name">
</div>
```

#### `v-if`

You can also use `v-if` to show or hide an element.  However, this directive will not use CSS.  It will either render the element or not.

```html
<div id="app">
	<h1 v-if="!name">What's your name?</h1>
	<h1 v-if="name">Hi, {{ name }}!</h1>
	<input v-model="name">
</div>
```

If you use the `<template>` tag with `v-if` and the data value is truthy, the contents will be rendered, but the template tag will not:

```html
<div id="app">
	<h1 v-if="!name">What's your name?</h1>
	<template v-if="name">
		<h1>Hi, {{ name }}!</h1>
		<p>It's nice to meet you.</p>
	</template>
	<input v-model="name">
</div>
```

You can also use standard conditional operators in the `v-if` and `v-show` directives:

```html
<h1 v-show="name">
	Hello, 
	<span v-if="gender == 'female'">miss</span>
	<span v-if="gender == 'male'">mister</span>
	{{ name }}.
</h1>
```

#### `v-else`

Following an element with a `v-if` directive, you can add an element with a `v-else` directive:

```html
<div id="app">
	<template v-if="name">
		<h1>Hi, {{ name }}!</h1>
		<p>It's nice to meet you.</p>
	</template>
	<template v-else>
		<h1>What's your name?</h1>
	</template>
	<input v-model="name">
</div>
```

#### `v-if` vs `v-show`

From the documentation:

> When using v-if, if the condition is false on initial render, it will not do anything - - the conditional block won’t be rendered until the condition becomes true for the first time. Generally speaking, v-if has higher toggle costs while v-show has higher initial render costs. So prefer v-show if you need to toggle something very often, and prefer v-if if the condition is unlikely to change at runtime.

#### `v-for`

Iterate a specified number of times:

```html
<ul>
	<li v-for="i in 11" class="list-group-item"> 
  		{{ i-1 }} times 4 equals {{ (i-1) * 4 }}.
	</li>
</ul>
```

Iterate through an array of strings:

```html
<ul>
	<li v-for="movie in movies">
		{{ movie }}
	</li>
</ul>
```

Iterate through an array of objects:

```html
<ul>
	<li v-for="movie in movies">
		{{ movie.title }} - {{ movie.release_date }}
	</li>
</ul>
```

Iterate through an array of objects with an index:

```html
<ul>
	<li v-for="(movie, index) in movies">
		{{ index }}. {{ movie.title }} - {{ movie.release_date }}
	</li>
</ul>
```

## Events and `v-on`

This is how you handle events on elements.  `v-on` is followed by the type of event, like `click` and the attribute value is the functioanality to execute when the event happens:

```html
<html>
	<head>
		<title>v-on</title>
		<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet">
	</head>
	<body>
		<div class="container">
			<div class="row">
				<div class="col-md-12">
					<br>
					<button v-on:click="upvotes++" class="btn btn-primary">
						Upvote {{ upvotes }}
					</button>
				</div>
			</div>
		</div>
	</body>
	<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.js"></script>
	<script>
		new Vue({
			el: '.container',
			data: { upvotes: 0 }
		});
	</script>
</html>
```

You can also call a method on the Vue object.  Add the method to a `methods` hash.

```html
<html>
	<head>
		<title>v-on</title>
		<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet">
	</head>
	<body>
		<div class="container">
			<div class="row">
				<div class="col-md-12">
					<br>
					<button v-on:click="upvote" class="btn btn-primary">
						Upvote {{ upvotes }}
					</button>
				</div>
			</div>
		</div>
	</body>
	<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.js"></script>
	<script>
		new Vue({
			el: '.container',
			data: { upvotes: 0 },
			methods: {
				upvote: function() {
					this.upvotes++;
				}
			}
		});
	</script>
</html>
```

The `v-on` directive has a shorthand:

```html
<button @click="upvote">Upvotes {{ upvotes }}</button>
```

The method receiving the event will have the `event` argument passed to it:

```javascript
// ...
upvote: function(event) {
	// ...
	event.preventDefault();
}
```

You can achieve the `event.preventDefault()` approach with the `.prevent` modifier on the `v-on` directive:

```html
<button @click.prevent="upvote">Upvotes {{ upvotes }}</button>
```

The `keyup` event also has identifiers that allow you to specify which key you care about.  Here we only care about the `enter` key:

```html
<input @keyup.enter="doSomething">
```

## Computed values

You can create computed values that behave like normal data properties:

```html
<html>
	<head>
		<title>v-on</title>
		<link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css" rel="stylesheet">
	</head>
	<body>
		<div class="container">
			<div class="row">
				<div class="col-md-12">
					<h1>Square a number</h1>
					<h4>Enter the number:</h4>
					<input v-model="number">
					<p>{{ number }} * {{ number }} = {{ squared }}</p>
				</div>
			</div>
		</div>
	</body>
	<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.0.1/vue.js"></script>
	<script>
		new Vue({
			el: '.container',
			data: { 
				number: 0 
			},
			computed: {
				squared: function() {
					return this.number * this.number;
				}
			}
		});
	</script>
</html>
```

## Adding Sass (scss) to webpack template

```bash
$ npm install node-sass sass-loader -D
```

After this is in place, you should be able to just use it in your components:

```html
<style lang="scss">
/* some scss */
</style>
```

## Debugging with Karma

First you need to load the Chrome Karma launcher:

```bash
$ npm install karma-chrome-launcher --save-dev
```

Then run the tests in debug mode with the browser set to Chrome:

```bash
$ ./node_modules/karma/bin/karma start test/unit/karma.conf.js --browser=Chrome --single-run=false --debug
```

In the browser, click the debug button, which will open a debug page.  Open the developer tools on this page, and you might need to reload.

You should now be able to debug stuff by sticking a `debugger` statement in your code and refreshing the karma debug page.
