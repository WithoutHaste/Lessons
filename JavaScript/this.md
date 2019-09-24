# The "this" keyword

`this` is a keyword in JavaScript.

`this` is also an environment variable. In other words, a variable provided by the environment automatically.

`this` refers to the executing context of the current line of code.

## Objects

In this example, we have an object that contains one property and one method.  The method needs to access the property.  We use the `this` variable to access the `name` property of the current object.

```
<html>
	<body>
	</body>
</html>

<script type='text/javascript'>

let customer = {
	name: 'Bob',
	displayGreeting: function() {
		console.log(`Greetings to ${this.name}.`);
	}
};

customer.displayGreeting(); //outputs: Greetings to Bob.

</script>
```
_(This style of string formatting is called a Template Literal.)_

## Event Handlers

## Callbacks