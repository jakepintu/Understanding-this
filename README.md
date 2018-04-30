# UNDERSTANDING `this`

## Introduction

One of the most confused mechanisms in JavaScript is the `this` keyword. It's a special identifier keyword that's automatically defined in the scope of every function, but what exactly it refers to bedevils even seasoned JavaScript developers.

## Why `this`
If the `this` mechanism is so confusing, even to seasoned JavaScript developers, one may wonder why it's even useful? Is it more trouble than it's worth? There isn't a single word that describes `this` well, so I just think of it as a special variable that changes depending on the situation. Those different situations are captured below.

## Case 1: 
### In a regular function (or if you're not in a function at all), `this` points to `window`. This is the default case.

```javascript
function logThis() {
  console.log(this);
}

logThis(); // window

// In strict mode, `this` will be `undefined` instead of `window`. 
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
```



## Case 2:
### When a function is called as a method, `this` points to the object that's on the left side of the dot.

```javascript

/*
 * You can also think of this as the "left of the dot" rule. 
 * For example, in myObject.myMethod(), `this` will be myObject
 * because myObject is to the left of the dot.
 *
 * Of course, if you're using this syntax myObject['myMethod'](),
 * technically it would be the "left of the dot or bracket" rule,
 * but that sounds clumsy and generally terrible.
 *
 * If you have multiple dots, the relevant dot is the one closest 
 * to the method call. For example, if you have one.two.hi();
 * `this` inside of hi will be two.
 */

var myObject = {
  myMethod: function() {
    console.log(this);
  }
};

myObject.myMethod(); // myObject

```

First, know that all functions in JavaScript have properties, just as objects have properties. And when a function executes, it gets the this property—a variable with the value of the object that invokes the function where this is used.

Ruminate on this basic example illustrating the use of this in JavaScript:

```javascript
    var person = {
    firstName   :"Penelope",
    lastName    :"Barrymore",
    
    // Since the "this" keyword is used inside the showFullName method below, and the showFullName method is defined on the person // object,​
     // "this" will have the value of the person object because the person object will invoke showFullName ()​
     
    showFullName:function () {
    console.log (this.firstName + " " + this.lastName);
    }
​
    }
​
    person.showFullName (); // Penelope Barrymore
```

## Its Scope
The next most common misconception about the meaning of `this` is that it somehow refers to the function's scope. It's a tricky question, because in one sense there is some truth, but in the other sense, it's quite misguided.

To be clear, `this` does not, in any way, refer to a function's <strong>lexical scope</strong>.It is true that internally, scope is kind of like an object with properties for each of the available identifiers. But the scope "object" is not accessible to JavaScript code. It's an inner part of the Engine's implementation.

Consider code which attempts (and fails!) to cross over the boundary and use `this` to implicitly refer to a function's lexical scope:

```javascript
function foo() {
	var a = 2;
	this.bar();
}

function bar() {
	console.log( this.a );
}

foo(); //undefined
```
There's more than one mistake in this snippet. While it may seem contrived, the code you see is a distillation of actual real-world code that has been exchanged in public community help forums. It's a wonderful (if not sad) illustration of just how misguided `this` assumptions can be.

Firstly, an attempt is made to reference the `bar()` function via `this.bar()`. It is almost certainly an accident that it works, but we'll explain the how of that shortly. The most natural way to have invoked `bar()` would have been to omit the leading `this`. and just make a lexical reference to the identifier.

However, the developer who writes such code is attempting to use `this` to create a bridge between the lexical scopes of `foo()` and `bar()`, so that `bar()` has access to the variable a in the inner scope of `foo()`. No such bridge is possible. You cannot use a this reference to look something up in a lexical scope. It is not possible.

Every time you feel yourself trying to mix lexical scope look-ups with `this`, remind yourself: there is no bridge.

## The use of `this` in the global scope
In the global scope, when the code is executing in the browser, all global variables and functions are defined on the window object. Therefore, when we use this in a global function, it refers to (and has the value of) the global window object (not in strict mode though, as noted earlier) that is the main container of the entire JavaScript application or web page.

Thus:

```javascript
    var firstName = "Peter",
    lastName = "Ally";
​
    function showFullName () {
    
    // "this" inside this function will have the value of the window object​
    // because the showFullName () function is defined in the global scope, just like the firstName and lastName​ 
    
    console.log (this.firstName + " " + this.lastName);
    }
​
    var person = {
    firstName   :"Penelope",
    lastName    :"Barrymore",
    showFullName:function () {
    
    // "this" on the line below refers to the person object, because the showFullName function will be invoked by person object.​
    
    console.log (this.firstName + " " + this.lastName);
    }
    }
​
    showFullName (); // Peter Ally​
​
    // window is the object that all global variables and functions are defined on, hence:​
    
    window.showFullName (); // Peter Ally​
​
    // "this" inside the showFullName () method that is defined inside the person object still refers to the person object, hence:​
    
    person.showFullName (); // Penelope Barrymore
```


## Case 3:
### In a function that's being called as a constructor, `this` points to the object that the constructor is creating.

```javascript
function Person(name) {
  this.name = name;
}

var jake = new Person('jake');
console.log(jake); // {name: 'jake'}
```



## Case 4:
### When you explicitly set the value of `this` manually using `bind`, `apply`, or `call`, it's all up to you.

```javascript
function logThis() {
  console.log(this);
}

var explicitlySetLogThis = logThis.bind({name: 'Jake'});

explicitlySetLogThis(); // {name: 'Jake'}

// Note that a function returned from .bind (like `boundOnce` below),
// cannot be bound to a different `this` value ever again.
// In other words, functions can only be bound once.
var boundOnce = logThis.bind({name: 'The first time is forever'});

// These attempts to change `this` are futile.
boundOnce.bind({name: 'why even try?'})();
boundOnce.apply({name: 'why even try?'});
boundOnce.call({name: 'why even try?'});
```



## Case 5:
### In a callback function, apply the above rules methodically.

```javascript

function outerFunction(callback) {
  callback();
}

function logThis() {
  console.log(this);
}

/*
 * Case 1: The regular old default case.
 */
 
outerFunction(logThis); // window

/*
 * Case 2: Call the callback as a method
 * (You'll probably NEVER see this, but I guess it's possible.)
 */
 
function callAsMethod(callback) {
  var weirdObject = {
    name: "Don't do this in real life"
  };
  
  weirdObject.callback = callback;
  weirdObject.callback();
}

callAsMethod(logThis); // `weirdObject` will get logged to the console

/*
 * Case 3: Calling the callback as a constructor. 
 * (You'll also probably never see this. But in case you do...)
 */
 
function callAsConstructor(callback) {
  new callback();
}

callAsConstructor(logThis); // the new object created by logThis will be logged to the console

/*
 * Case 4: Explicitly setting `this`.
 */
 
function callAndBindToJake(callback) {
  var boundCallback = callback.bind({name: 'Jake'});
  boundCallback();
}

callAndBindToJake(logThis); // {name: 'Jake'}

// In a twist, we give `callAndBindToJake` a function that's already been bound.
var boundOnce = logThis.bind({name: 'The first time is forever'});
callAndBindToJake(boundOnce); // {name: 'The first time is forever'}
```

## Conclusion
I am hopeful you have learned enough to help you understand the this keyword in JavaScript. Now you have the tools (bind, apply, and call, and setting this to a variable) necessary to conquer JavaScript’s this in every scenario.

As you have learned, this gets a bit troublesome in situations where the original context (where this was defined) changes, particularly in callback functions, when invoked with a different object, or when borrowing methods. Always remember that this is assigned the value of the object that invoked the this Function.
