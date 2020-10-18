
# Class basic syntax

```quote author="উইকিপিডিয়া"
অবজেক্ট ওরিয়েন্টেড প্রোগ্রামিং এ, *class* হলো এমন একটি কোড টেম্পলেট যেটি কিছু ভ্যরিয়াবল ও মেথড এর শুরুর অবস্থা ও আচরণ ধরে রাখে এবং এটিকে পরবর্তীতে বর্ধিত করে ব্যবহার করা যায়।
```

ব্যবহারিক ক্ষেত্রে আমাদের প্রায়ই একই ধরনের অবজেক্ট তৈরি করতে হয়। যেমনঃ users, goods বা অন্যকিছু।

<info:constructor-new>, এই চ্যাপ্টার থেকে আমরা ইতোমধ্যে জানি যে, `new function` দিয়েও আমরা এই কাজটি করতে পারি।

কিন্তু মডার্ন জাভাস্ক্রিপ্টে আরো এডভান্স "class" রয়েছে, যা আমাদের আরো অনেক ভালো ও দরকারি ফিচার এর সাথে পরিচয় করিয়েছে। যা অবজেক্ট ওরিয়েন্টেড প্রোগ্রামিং এ খুবই দরকারি।

## "class" এর গঠন

মুল গঠন:
```js
class MyClass {
  // class methods
  constructor() { ... }
  method1() { ... }
  method2() { ... }
  method3() { ... }
  ...
}
```

এখন আমরা `new MyClass()` লিখে নতুন অবজেক্ট তৈরি করতে পারি ভেতরের মেথড গুলো সহ।

 `constructor()` মেথডটি `new` দ্বারা স্বয়ংক্রিয় ভাবে কল হয়, তাই আমরা এইখানে অবজেক্ট শুরু করতে পারি। 

যেমন:

```js run
class User {

  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert(this.name);
  }

}

// ব্যবহার:
let user = new User("John");
user.sayHi();
```

যখন `new User("John")` ডাকা হয়:
1. একটি নতুন অবজেক্ট তৈরি হয়।
2. `constructor` টি আর্গুমেন্ট নিয়ে রান করে এবং `this.name` এ আর্গুমেন্ট টি এসাইন করে দেয়।

...Then we can call object methods, such as `user.sayHi()`.


```warn header="No comma between class methods"
A common pitfall for novice developers is to put a comma between class methods, which would result in a syntax error.

The notation here is not to be confused with object literals. Within the class, no commas are required.
```

## What is a class?

So, what exactly is a `class`? That's not an entirely new language-level entity, as one might think.

Let's unveil any magic and see what a class really is. That'll help in understanding many complex aspects.

In JavaScript, a class is a kind of a function.

Here, take a look:

```js run
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

// proof: User is a function
*!*
alert(typeof User); // function
*/!*
```

What `class User {...}` construct really does is:

1. Creates a function named `User`, that becomes the result of the class declaration. The function code is taken from the `constructor` method (assumed empty if we don't write such method).
2. Stores class methods, such as `sayHi`, in `User.prototype`.

After `new User` object is created, when we call its method, it's taken from the prototype, just as described in the chapter <info:function-prototype>. So the object has access to class methods.

We can illustrate the result of `class User` declaration as:

![](class-user.svg)

Here's the code to introspect it:

```js run
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

// class is a function
alert(typeof User); // function

// ...or, more precisely, the constructor method
alert(User === User.prototype.constructor); // true

// The methods are in User.prototype, e.g:
alert(User.prototype.sayHi); // alert(this.name);

// there are exactly two methods in the prototype
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

## Not just a syntax sugar

Sometimes people say that `class` is a "syntax sugar" (syntax that is designed to make things easier to read, but doesn't introduce anything new), because we could actually declare the same without `class` keyword at all:

```js run
// rewriting class User in pure functions

// 1. Create constructor function
function User(name) {
  this.name = name;
}
// any function prototype has constructor property by default,
// so we don't need to create it

// 2. Add the method to prototype
User.prototype.sayHi = function() {
  alert(this.name);
};

// Usage:
let user = new User("John");
user.sayHi();
```

The result of this definition is about the same. So, there are indeed reasons why `class` can be considered a syntax sugar to define a constructor together with its prototype methods.

Still, there are important differences.

1. First, a function created by `class` is labelled by a special internal property `[[FunctionKind]]:"classConstructor"`. So it's not entirely the same as creating it manually.

    And unlike a regular function, a class constructor must be called with `new`:

    ```js run
    class User {
      constructor() {}
    }

    alert(typeof User); // function
    User(); // Error: Class constructor User cannot be invoked without 'new'
    ```

    Also, a string representation of a class constructor in most JavaScript engines starts with the "class..."

    ```js run
    class User {
      constructor() {}
    }

    alert(User); // class User { ... }
    ```

2. Class methods are non-enumerable.
    A class definition sets `enumerable` flag to `false` for all methods in the `"prototype"`.

    That's good, because if we `for..in` over an object, we usually don't want its class methods.

3. Classes always `use strict`.
    All code inside the class construct is automatically in strict mode.

Besides, `class` syntax brings many other features that we'll explore later.

## Class Expression

Just like functions, classes can be defined inside another expression, passed around, returned, assigned etc.

Here's an example of a class expression:

```js
let User = class {
  sayHi() {
    alert("Hello");
  }
};
```

Similar to Named Function Expressions, class expressions may have a name.

If a class expression has a name, it's visible inside the class only:

```js run
// "Named Class Expression"
// (no such term in the spec, but that's similar to Named Function Expression)
let User = class *!*MyClass*/!* {
  sayHi() {
    alert(MyClass); // MyClass name is visible only inside the class
  }
};

new User().sayHi(); // works, shows MyClass definition

alert(MyClass); // error, MyClass name isn't visible outside of the class
```


We can even make classes dynamically "on-demand", like this:

```js run
function makeClass(phrase) {
  // declare a class and return it
  return class {
    sayHi() {
      alert(phrase);
    };
  };
}

// Create a new class
let User = makeClass("Hello");

new User().sayHi(); // Hello
```


## Getters/setters, other shorthands

Just like literal objects, classes may include getters/setters, computed properties etc.

Here's an example for `user.name` implemented using `get/set`:

```js run
class User {

  constructor(name) {
    // invokes the setter
    this.name = name;
  }

*!*
  get name() {
*/!*
    return this._name;
  }

*!*
  set name(value) {
*/!*
    if (value.length < 4) {
      alert("Name is too short.");
      return;
    }
    this._name = value;
  }

}

let user = new User("John");
alert(user.name); // John

user = new User(""); // Name too short.
```

The class declaration creates getters and setters in `User.prototype`, like this:

```js
Object.defineProperties(User.prototype, {
  name: {
    get() {
      return this._name
    },
    set(name) {
      // ...
    }
  }
});
```

Here's an example with a computed property in brackets `[...]`:

```js run
class User {

*!*
  ['say' + 'Hi']() {
*/!*
    alert("Hello");
  }

}

new User().sayHi();
```

## Class properties

```warn header="Old browsers may need a polyfill"
Class-level properties are a recent addition to the language.
```

In the example above, `User` only had methods. Let's add a property:

```js run
class User {
*!*
  name = "Anonymous";
*/!*

  sayHi() {
    alert(`Hello, ${this.name}!`);
  }
}

new User().sayHi();
```

The property `name` is not placed into `User.prototype`. Instead, it is created by `new` before calling the constructor, it's a property of the object itself.

## Summary

The basic class syntax looks like this:

```js
class MyClass {
  prop = value; // property

  constructor(...) { // constructor
    // ...
  }

  method(...) {} // method

  get something(...) {} // getter method
  set something(...) {} // setter method

  [Symbol.iterator]() {} // method with computed name (symbol here)
  // ...
}
```

`MyClass` is technically a function (the one that we provide as `constructor`), while methods, getters and settors are written to `MyClass.prototype`.

In the next chapters we'll learn more about classes, including inheritance and other features.
