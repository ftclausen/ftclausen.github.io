---
title: "Function currying example that works for me"
date: 2018-07-25 14:00
classes: wide
categories: general
---

This example helps me understand one use case of function currying - pre-applying a common argument to make code more
succinct. These are two utility functions: `method` bolted onto the Function prototype and a `curry` method added to
objects via the `method` function. To see currying in action take a look from `sprayPaintCar()` onwards.

Without further ado:

```javascript
// Helper method taken from JS: The Good Parts
Function.prototype.method = function(name, func) {
  this.prototype[name] = func;
  return this;
};

// Bolt a 'curry' method onto the Function prototype that retrofits the slice method to the "arguments" array-like
object. We then can combine the arguments from the original function with the newly defined "pre-filled" function.
Function.method('curry', function() {
  var slice = Array.prototype.slice,
      args = slice.apply(arguments),
      that = this;
  return function() {
    return that.apply(null, args.concat(slice.apply(arguments)));
  };
});

// Actual function we want to use in this currying example
var sprayPaintCar = function(make, colour) {
  return "Car make " + make + " has been spray painted " + colour + " colour.";
}

console.log(sprayPaintCar('toyota', 'red'));
// By using currying, if we are only spray painting Toyota cars then we can create a curried function for convenience.
sprayPaintAToyota = sprayPaintCar.curry('toyota');
console.log(sprayPaintAToyota('blue'));
console.log(sprayPaintAToyota('pink'));
``` 
