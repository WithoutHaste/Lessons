Link: www.GitHub.com/WithoutHaste/Lessons/JavaScript/this.md

# The "this" keyword

`this` is an environment variable in JavaScript. 

Whenever you execute a function, `this` is implicitly bound to the object that is executing the current function.  
_'Bound' means 'set equal to'._

You can say that `this` is bound to the executing object.  
You can say that `this` is bound to the executing context.

## Functions vs Methods

A method is attached to an object.
```
let customer = {
	name: 'Bob',
	getName: function() {
		return this.name;
	}
};
```

A function is not attached to an object. It is defined in the global scope, within another function or method, or is passed as a parameter.
```
function doSomething() {
	console.log('something');
	
	function innerDoSomething() {
		console.log('inner something');
	}
}
```

```
function runCallback(callback) {
	callback();
}

runCallback(function() {
	console.log('something');
});
```

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
_(This style of string formatting, with the `${}`, is called a Template Literal.)_

If we didn't use the `this` variable inside `displayGreeting`, the result would be `Greetings to .` because there is no `name` variable in the scope of that function.

This example works because the `this` of the function is automatically bound to the executing object `customer`.

It does not matter where a function is defined, it only matters where it is executed from:
```
function globalDisplayGreeting() {
	console.log(`Greetings to ${this.name}.`);
}

let customer = {
	name: 'Bob',
	displayGreeting: globalDisplayGreeting
}

customer.displayGreeting(); //outputs: Greetings to Bob.
```

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

The button is an object, and we've added a function (the event handler) to that object.  `this` inside the event handler is automatically bound to the executing object `button`.

## Global Functions

Global functions are functions defined in the global scope.  The executing object of all global functions is the global object, also called the `window`.

```
function isWindow(object) {
	return (object == window);
}

function doSomething() {
	console.log('Doing something.');
	console.log(`this equals the window: ${isWindow(this)}`);
}

doSomething();

//outputs: 
//  Doing something.
//  this equals the window: true
```

When you use nested functions, the inner functions will still run in the global context.

In this example, we are explicitly binding `doSomething`'s `this` to the object `bob`.  Note that `innerDoSomething` still has its `this` bound to the `window`.

```
let bob = {
	name: 'Bob'
};

function isWindow(object) {
	return (object == window);
}

function doSomething() {
	console.log('Doing something.');
	console.log(`this is ${this.name}`);
	innerDoSomething();
	
	function innerDoSomething() {
		console.log('Inner doing something.');
		console.log(`this equals the window: ${isWindow(this)}`);
		console.log(this.name);
	}
}

doSomething.call(bob);

//outputs: 
//  Doing something.
//  this is Bob
//  Inner doing something.
//  this equals the window: true
```

## Function as constructor

If you use the `new` keyword on a function call, you instantiate a new object.  

`this` inside a constructor is bound to the new object being instantiated.

```
function customer(name) {
	this.name = name;
	this.displayGreeting = function() {
		console.log(`Greetings to ${this.name}.`);
	};
}

let bob = new customer('Bob');

bob.displayGreeting(); //outputs: Greetings to Bob.
```

## Callbacks

A callback is a function (A) that is passed into another function (B), to be run at the end of function (B).  This is where `this` gets more complicated.

Here's a simple example of a callback:
```
function doSomething(callback) {
	console.log('Doing something.');
	callback();
}

doSomething(function() {
	console.log('Finished.');
});

//outputs: 
//  Doing something.
//  Finished.
```

Remember that in global functions, `this` is just bound to the `window` object, which is not very useful.

Let's use objects instead, so we see that happens with `this`.

```
function isWindow(object) {
	return (object == window);
}

let customer = {
	name: 'Bob',
	test: function(callback) {
		console.log('Inside customer object.');
		console.log(`this.name is ${this.name}`);
		callback();
	}
};

let company = {
	name: 'Google',
	test: function() {
		console.log('Inside company object.');
		console.log(`this.name is ${this.name}`);
		customer.test(this.callback);
	},
	callback: function() {
		console.log('Inside callback function.');
		console.log(`this equals the window: ${isWindow(this)}`);
	}
}

company.test();

//outputs: 
//  Inside company object.
//  this.name is Google
//  Inside customer object.
//  this.name is Bob
//  Inside callback function.
//  this equals the window: true
```

You might have guessed that `Inside callback function` would have been followed by another `this.name is Google`. But instead, we got another `this equals the window: true`. 

That's because `this` is determined by where a function is executed, not by where it was defined. As with the nested function example earlier, the inner function has been executed in the global scope.

__The callback problem: callbacks, by their nature, are executed by a different context than defines them. How can they access the `this` value we expect them to?__

### Arrow Functions

The callback problem is the main reason arrow functions were added to JavaScript.

Arrow functions do not bind `this` to the object that __executes__ them.  
Arrow functions bind `this` to the object that __defined__ them.  

Keep this behavior difference in mind anytime you use arrow functions.  This behavior is different from all other ways of defining functions.

If we alter the previous example to use arrow functions, we'll see the results we wanted.
```
let customer = {
	name: 'Bob',
	test: function(callback) {
		console.log('Inside customer object.');
		console.log(`this.name is ${this.name}`);
		callback();
	}
};

let company = {
	name: 'Google',
	test: function() {
		console.log('Inside company object.');
		console.log(`this.name is ${this.name}`);
		customer.test(() => this.callback()); //here's the arrow function
	},
	callback: function() {
		console.log('Inside callback function.');
		console.log(`this.name is ${this.name}`);
	}
}

company.test();

//outputs: 
//  Inside company object.
//  this.name is Google
//  Inside customer object.
//  this.name is Bob
//  Inside callback function.
//  this.name is Google
```

Timeouts are a common reason to use an arrow function:
```
let customer = {
	name: 'Bob',
	delayedGreeting: function() {
		setTimeout(
			() => console.log(`Greetings to ${this.name}.`),
			3000
		);
	}
};

customer.delayedGreeting(); //outputs: Greetings to Bob.
```
We use an arrow function here because the callback we define will be executed from the `setTimeout` event handler. If we want access to the object that set the timeout, we must use the special `this` binding behavior of arrow functions.

Try it with a normal function to see the difference:
```
let customer = {
	name: 'Bob',
	delayedGreeting: function() {
		setTimeout(
			function() { console.log(`Greetings to ${this.name}.`); },
			3000
		);
	}
};

customer.delayedGreeting(); //outputs: Greetings to .
```

## Explicit bindings

You can override the implicit binding of `this` with an explicit binding.

### call()

`call` will execute a function and will explicitly bind its `this` to whatever object you provide.

It also accepts arguments for the function.

```
let customerBob = {
	name: 'Bob',
	displayGreeting: function(company) {
		console.log(`Greetings to ${this.name} from ${company}`);
	}
}
let customerJane = {
	name: 'Jane',
	displayGreeting: function(company) {
		console.log(`Greetings to ${this.name} from ${company}`);
	}
}

customerBob.displayGreeting.call(customerJane, 'Google'); //outputs: Greetings to Jane from Google
```

### apply()

`apply` will execute a function and will explicitly bind its `this` to whatever object you provide.

It also accepts an array of arguments for the function.

```
let customerBob = {
	name: 'Bob',
	displayGreeting: function(company) {
		console.log(`Greetings to ${this.name} from ${company}`);
	}
}
let customerJane = {
	name: 'Jane',
	displayGreeting: function(company) {
		console.log(`Greetings to ${this.name} from ${company}`);
	}
}

customerBob.displayGreeting.apply(customerJane, ['Google']); //outputs: Greetings to Jane from Google
```

### bind()

`bind` will return a new function. When you run the new function, its `this` will be explicitly bound to whatever object you provided.

```
let customerBob = {
	name: 'Bob',
	displayGreeting: function() {
		console.log(`Greetings to ${this.name}`);
	}
}
let customerJane = {
	name: 'Jane',
	displayGreeting: function() {
		console.log(`Greetings to ${this.name}`);
	}
}

let janeDisplayGreeting = customerBob.displayGreeting.bind(customerJane);

janeDisplayGreeting(); //outputs: Greetings to Jane
```

### Hard Binding

Hard binding is a trick to ensure that every time function (A) is called, it always has `this` bound to the same object.

```
function displayGreeting() {
	console.log(`Greetings to ${this.name}`);
}

let customerBob = { 
	name: 'Bob'
};
let customerJane = { 
	name: 'Jane'
};

let temp = displayGreeting;
displayGreeting = function() { 
	temp.call(customerBob); //this is the hard binding
};

displayGreeting(); //outputs: Greetings to Bob

displayGreeting.call(customerJane); //outputs: Greetings to Bob
```
