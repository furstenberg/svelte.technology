---
title: Element directives
---

Directives are element or component-level instructions to Svelte. They look like attributes, except with a `:` character.

### Event handlers

In most applications, you'll need to respond to the user's actions. In Svelte, this is done with the `on:[event]` directive.

```html
<p>Count: {{count}}</p>
<button on:click='set({ count: count + 1 })'>+1</button>
```

```hidden-data
{
	"count": 0
}
```

When the user clicks the button, Svelte calls `component.set(...)` with the provided arguments. You can call any method belonging to the component (whether [built-in](#component-api) or [custom](#custom-methods)), and any data property (or computed property) that's in scope:

```html
<p>Select a category:</p>

{{#each categories as category}}
	<button on:click='select( category )'>select {{category}}</button>
{{/each}}

<script>
	export default {
		data () {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		},

		methods: {
			select ( name ) {
				alert( `selected ${name}` );
			}
		}
	};
</script>
```

You can also access the `event` object in the method call:

```html
<div on:mousemove='set({ x: event.clientX, y: event.clientY })'>
	coords: {{x}},{{y}}
</div>

<style>
	div {
		border: 1px solid purple;
		width: 100%;
		height: 100%;
	}
</style>
```

The target node can be referenced as `this`, meaning you can do this sort of thing:

```html
<input on:focus='this.select()'>
```

### Custom events

You can define your own custom events to handle complex user interactions like dragging and swiping. See the earlier section on [custom event handlers](#custom-event-handlers) for more information.

### Component events

Events are an excellent way for [nested components](#nested-components) to communicate with their parents. Let's revisit our earlier example, but turn it into a `<CategoryChooser>` component:

```html-no-repl
<!-- CategoryChooser.html -->
<p>Select a category:</p>

{{#each categories as category}}
	<button on:click='fire("select", { category })'>select {{category}}</button>
{{/each}}

<script>
	export default {
		data() {
			return {
				categories: [
					'animal',
					'vegetable',
					'mineral'
				]
			}
		}
	};
</script>
```

When the user clicks a button, the component will fire a `select` event, where the `event` object has a `category` property. Any component that nests `<CategoryChooser>` can listen for events like so:

```html-no-repl
<CategoryChooser on:select='playTwentyQuestions(event.category)'/>

<script>
	import CategoryChooser from './CategoryChooser.html';

	export default {
		components: {
			CategoryChooser
		},

		methods: {
			playTwentyQuestions (category) {
				// TODO implement
			}
		}
	};
</script>
```

Just as `this` in an element's event handler refers to the element itself, in a component event handler `this` refers to the component firing the event.


### Refs

Refs are a convenient way to store a reference to particular DOM nodes or components. Declare a ref with `ref:[name]`, and access it inside your component's methods with `this.refs.[name]`:

```html
<canvas ref:canvas width='200' height='200'></canvas>

<script>
	export default {
		oncreate () {
			const canvas = this.refs.canvas;
			const ctx = canvas.getContext( '2d' );

			let destroyed = false;
			this.on( 'destroy', () => destroyed = true );

			function loop () {
				if ( destroyed ) return;
				requestAnimationFrame( loop );

				const imageData = ctx.getImageData( 0, 0, canvas.width, canvas.height );

				for ( let p = 0; p < imageData.data.length; p += 4 ) {
					const i = p / 4;
					const x = i % canvas.width;
					const y = i / canvas.height >>> 0;

					const t = window.performance.now();

					const r = 64 + ( 128 * x / canvas.width ) + ( 64 * Math.sin( t / 1000 ) );
					const g = 64 + ( 128 * y / canvas.height ) + ( 64 * Math.cos( t / 1000 ) );
					const b = 128;

					imageData.data[ p + 0 ] = r;
					imageData.data[ p + 1 ] = g;
					imageData.data[ p + 2 ] = b;
					imageData.data[ p + 3 ] = 255;
				}

				ctx.putImageData( imageData, 0, 0 );
			}

			loop();
		}
	}
</script>
```

> Since only one element or component can occupy a given `ref`, don't use them in `{{#each ...}}` blocks. It's fine to use them in `{{#if ...}}` blocks however.


### Two-way binding

It's currently fashionable to avoid two-way binding on the grounds that it creates all sorts of hard-to-debug problems and slows your application down, and that a one-way top-down data flow is 'easier to reason about'. This is in fact high grade nonsense. It's true that two-way binding done *badly* has all sorts of issues, and that very large apps benefit from the discipline of a not permitting deeply nested components to muck about with state that might affect distant parts of the app. But when used correctly, two-way binding simplifies things greatly.

Bindings are declared with the `bind:[attribute]` directive:

```html
<input bind:value='name' placeholder='enter your name'>
<p>Hello {{name || 'stranger'}}!</p>
```

Here are the current bindable attributes and properties for each element:

- `<input>`, `<textarea>`, `<select>`, `<option>`
	- `value`
- `<input type="checkbox">`, `<input type="radio">`
	- `group`
- `<audio>`, `<video>`
	- `buffered`, `currentTime`, `duration`, `paused`, `played`, `seekable`

As well as DOM elements, you can bind to component data properties:

```html-no-repl
<CategoryChooser bind:category='category'/>
```

If the attribute and the bound property share a name, you can use this shorthand:

```html-no-repl
<CategoryChooser bind:category/>
```

Here is a complete example of using two way bindings with a form:

```html
<form on:submit='handleSubmit( event )'>
	<input bind:value='test' type='text'>
	<button type='submit'>Store</button>
</form>

<script>
export default {
	methods: {
		handleSubmit: function ( event ) {
			// prevent the page from reloading
			event.preventDefault();

			var value = this.get( 'test' );
			console.log( 'value', value );
		}
	}
};
</script>
```
