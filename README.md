### Understanding `this`

There isn't a single word that describes `this` well, so I just think of it as a special variable that changes depending on the situation. Those different situations are captured below.

## Case 1: In a regular function (or if you're not in a function at all), `this` points to `window`. This is the default case.

```javascript
function logThis() {
  console.log(this);
}

logThis(); // window

// In strict mode, `this` will be `undefined` instead of `window`. 
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
```

## Case 2: When a function is called as a method, `this` points to the object that's on the left side of the dot.

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

## Case 3: In a function that's being called as a constructor, `this` points to the object that the constructor is creating.

```javascript
function Person(name) {
  this.name = name;
}

var jake = new Person('jake');
console.log(jake); // {name: 'jake'}
```

## Case 4: When you explicitly set the value of `this` manually using `bind`, `apply`, or `call`, it's all up to you.

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

## Case 5: In a callback function, apply the above rules methodically.

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
