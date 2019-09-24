# The "this" keyword

`this` is a keyword in JavaScript.

`this` is also an environment variable. In other words, a variable provided by the environment automatically.

`this` refers to the executing context of the current line of code.

## Objects

In this example, we have an object that contains one property and one method.  The method needs to access the property.  

We use the `this` variable to access the `name` property of the current object.

```
let customer = {
	name: 'Bob',
	displayGreeting: function() {
		console.log(`Greetings to ${this.name}.`);
	}
};

customer.displayGreeting(); //outputs: Greetings to Bob.
```
_(This style of string formatting, with the `$[]`, is called a Template Literal.)_

If we didn't use the `this` variable inside `displayGreeting`, the result would be `Greetings to .`.

## Event Handlers

In this example, we add an event handler to a button.  The event handler needs access to the HTML element that was clicked.

We use the `this` variable to access the `customer` data attribute of the current HTML element.

```
<html>
	<body>
		<button id='button' data-customer='Bob'>Test</button>
	</body>
</html>

<script type='text/javascript'>

const button = document.getElementById('button');
button.addEventListener('click', function () {
	let customer = this.dataset.customer;
	console.log(`Clicked on ${customer}.`);  //outputs: Clicked on Bob.
});

</script>
```

## Callbacks