# Functions, Scope, Closures & `this`

Functions are the unit of composition in JavaScript. They are also objects, they carry scope, and they determine what `this` means at call time.

---

## 1. Function Forms

A function is a callable object. JavaScript gives you several syntaxes for creating one. They all produce callable functions, but they differ in **hoisting**, **`this`**, **`arguments`**, **constructability (`new`)**, and **when they become usable**. Knowing which is which prevents most "why is this `undefined`?" bugs.

### 1.1 Function declaration

The classic form: a named function defined as a statement.

```js
function greet(name) {
  return `Hello, ${name}`;
}

console.log(greet("Ada")); // "Hello, Ada"
```

**Fully hoisted.** The engine registers the whole function during the creation phase, so it can be called before its line:

```js
console.log(square(4)); // 16 — works before the declaration

function square(n) {
  return n * n;
}
```

Properties:

- Must have a name; the name is a `const`-like binding inside its scope.
- Has its own `this`, its own `arguments`, a `prototype` property, and can be called with `new`.
- Good for top-level utilities and anything you want to use before its definition.

### 1.2 Function expression

The function is created as a **value** and assigned to a variable. The variable is hoisted (`var`) or in the TDZ (`let`/`const`), but the function itself only exists once the assignment line runs.

```js
const greet2 = function (name) {
  return `Hello, ${name}`;
};

console.log(greet2("Grace")); // "Hello, Grace"
```

Not hoisted — calling before the assignment throws:

```js
try {
  console.log(cube(3));
} catch (error) {
  console.log(error.name); // "ReferenceError" — `cube` is in the TDZ
}

const cube = function (n) {
  return n ** 3;
};

console.log(cube(3)); // 27
```

With `var` the error changes, but it still fails:

```js
try {
  console.log(six());
} catch (error) {
  console.log(error.name); // "TypeError" — `six` is undefined, so it is not callable
}

var six = function () {
  return 6;
};
```

- Anonymous expressions are fine, but **naming** the expression helps recursion and stack traces:

```js
const factorial = function fact(n) {
  return n <= 1 ? 1 : n * fact(n - 1); // `fact` is visible only inside
};

console.log(factorial(5)); // 120
console.log(typeof fact);  // "undefined" — not visible outside
```

- Still a regular function: own `this`, own `arguments`, and constructable with `new`.

### 1.3 Arrow function

A shorter syntax for function expressions. It is **not** a replacement for every function — an arrow has no own `this`, no `arguments`, no `prototype`, and cannot be constructed.

```js
const double = (n) => n * 2;                  // one parameter: parentheses optional
const add = (a, b) => a + b;                  // two parameters: parentheses required
const now = () => Date.now();                 // zero parameters: parentheses required
const makeUser = (name) => ({ name, id: 1 }); // object literal: wrap the body in ()

console.log(double(5));       // 10
console.log(add(2, 3));       // 5
console.log(typeof now());    // "number"
console.log(makeUser("Ada")); // { name: "Ada", id: 1 }

// Expression body returns implicitly; a block body needs an explicit `return`
const withBlock = (n) => {
  const doubled = n * 2;
  return doubled + 1;
};

console.log(withBlock(4)); // 9
```

Without parentheses, `() => { name: "Ada" }` is a block with a label, not an object. That is why the parentheses around the object literal are required.

**No `arguments`** — use rest parameters instead:

```js
function withArguments() {
  console.log(arguments.length); // 3
}

const withArrow = () => {
  try {
    console.log(arguments.length); // ReferenceError at module scope
  } catch (error) {
    console.log(error.name); // "ReferenceError"
  }
};

withArguments("a", "b", "c");
withArrow(1, 2, 3);
```

**No own `this`** — it inherits `this` from where it is written:

```js
"use strict";

const team = {
  name: "Core",
  members: ["A", "B"],
  list() {
    // The arrow keeps `this` from list(), so this.name is "Core"
    return this.members.map((member) => `${this.name}: ${member}`);
  },
};

console.log(team.list()); // ["Core: A", "Core: B"]
```

That is why an arrow stored as an object property cannot see the object:

```js
"use strict";

const person = {
  name: "Ada",
  greetRegular() {
    return this.name; // regular method: `this` is person
  },
  greetArrow: () => `Hello, ${this?.name}`, // arrow: `this` is the enclosing scope
};

console.log(person.greetRegular()); // "Ada"
console.log(person.greetArrow());   // "Hello, undefined"
```

**Not a constructor, no `prototype`:**

```js
const Arrow = () => {};

try {
  new Arrow();
} catch (error) {
  console.log(error.name); // "TypeError" — arrows are not constructors
}

function Regular() {}
console.log(typeof Regular.prototype); // "object"
console.log(Arrow.prototype);          // undefined
```

A constructor needs an own `this` to attach properties to; an arrow does not have one.

### 1.4 Method shorthand

When a function lives on an object, use the shorthand. It is equivalent to `greet: function () {}`, just shorter.

```js
const user = {
  name: "Ada",
  greet() {
    return `Hello, ${this.name}`; // `this` is the object before the dot
  },
};

console.log(user.greet()); // "Hello, Ada"

// Long form, same behavior
const sameUser = {
  name: "Ada",
  greet: function () {
    return `Hello, ${this.name}`;
  },
};

console.log(sameUser.greet()); // "Hello, Ada"
```

- `this` is the **receiver** (`user`), not the lexical scope.
- Methods are not constructors and have no `prototype`:

```js
const user = {
  name: "Ada",
  greet() {
    return `Hello, ${this.name}`;
  },
};

try {
  new user.greet();
} catch (error) {
  console.log(error.name); // "TypeError" — shorthand methods cannot be constructed
}

console.log(user.greet.prototype); // undefined
```

- Class bodies use the same shorthand (see `03-objects-prototypes-classes.md`).

> **Gotcha:** Do **not** use an arrow as an object method when it needs the object's data. The arrow ignores the object and looks outward for `this`:

```js
"use strict";

const broken = {
  name: "Ada",
  greet: () => `Hello, ${this?.name}`, // `this` is NOT `broken`
};

console.log(broken.greet()); // "Hello, undefined"
```

An object literal creates no `this` of its own. The arrow inherits `this` from the surrounding code (the module, where it is `undefined`).

### 1.5 Immediately Invoked Function Expression (IIFE)

An IIFE is defined and called in the same expression. It runs once, and its variables stay private.

```js
const total = (function () {
  const price = 100;
  const tax = 0.2;
  return price + price * tax; // helpers are private
})();

console.log(total);        // 120
console.log(typeof price); // "undefined" — nothing leaked
```

Two equivalent syntaxes:

```js
(function () {
  console.log("parentheses outside");
})();

(function () {
  console.log("parentheses inside");
}());
```

The wrapping parentheses turn the function into an **expression**. Written bare, `function () {}` is a declaration and needs a name:

```js
// SyntaxError: function statements require a function name
// function () { console.log("nope"); }();
```

An arrow IIFE is shorter and is still used for async setup:

```js
(async () => {
  const message = await Promise.resolve("loaded");
  console.log(message); // "loaded"
})();
```

- Before ES modules, IIFEs prevented leaking globals.
- **Today**, prefer modules or block scope (`{ let x = 1; }`). Use an IIFE mainly for a one-off async entry point.
- **ASI gotcha:** if the previous line has no semicolon, a leading `(` can be parsed as a function call. Start the IIFE with `;`:

```js
const value = 1
;(function () {
  console.log("safe from ASI issues"); // "safe from ASI issues"
})();
```

### 1.6 What happens when you call a function

1. A new **execution context** is created: fresh scope, fresh `this`.
2. Arguments are bound: missing ones become `undefined`; extra ones are still reachable via `arguments`.
3. The body runs top to bottom.
4. The value after `return` is handed back — or `undefined` if there is no `return`.

```js
function noReturn(name) {
  const greeting = `Hello, ${name}`; // computed, but not returned
}

function explicitReturn(name) {
  return `Hello, ${name}`;
}

console.log(noReturn("Ada"));       // undefined
console.log(explicitReturn("Ada")); // "Hello, Ada"

function showArgs(a, b) {
  console.log(a, b, arguments.length);
}

showArgs("x", "y", "z"); // "x" "y" 3 — extra arguments are allowed
showArgs("x");           // "x" undefined 1 — missing arguments become undefined
```

> **Rule:** A function without `return` (or with a bare `return;`) always evaluates to `undefined`.

### 1.7 Which form should I use?

| Need | Use |
| :--- | :--- |
| Reusable named utility, callable before its line | Function declaration |
| A function stored in a variable or passed as a value | Function expression |
| Short callback; must inherit `this` from around it | Arrow function |
| Behavior attached to an object or class | Method shorthand |
| Run once, keep helpers private (legacy) | IIFE |

### 1.8 Arrow functions vs regular functions

| Feature | Regular function | Arrow function |
| :--- | :--- | :--- |
| Own `this` | Yes (set at call time) | No (lexical) |
| `arguments` object | Yes | No |
| Can be a constructor (`new`) | Yes | No |
| `prototype` property | Yes | No |
| Hoisted (as a declaration) | Yes | No |
| Implicit return | No | Yes (expression body) |

> **Rule:** Use arrow functions for callbacks that should inherit `this`. Use regular functions for object methods, constructors, and anything that needs `arguments`.

### 1.9 Arrows keep `this` in callbacks

A timer inside an object is the classic test. The arrow inherits `this` from `start()`:

```js
const timer = {
  seconds: 0,
  start() {
    const id = setInterval(() => {
      this.seconds++; // `this` is `timer`
      console.log(`tick ${this.seconds}`);
      if (this.seconds === 3) clearInterval(id);
    }, 1000);
  },
};

timer.start();
// tick 1
// tick 2
// tick 3
```

With `function () { ... }` instead, the callback would get its own `this` (`undefined` in a module), and `this.seconds++` would throw. If you must use a regular callback, bind it: `setInterval(function () { ... }.bind(this), 1000)`.

---

## 2. Parameters

```js
// Default parameters
function createUser(name, role = "member") {
  return { name, role };
}

// Rest parameters (always last)
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

// Destructured parameters
function render({ title, body, tags = [] }) {
  return `${title}: ${body} (${tags.join(", ")})`;
}
```

> **Gotcha:** Default values are evaluated at call time, so mutable defaults can leak state:

```js
function addItem(item, list = []) {
  list.push(item); // a new array is created per call because the default runs each time
  return list;
}
```

---

## 3. Scope

JavaScript has four kinds of scope:

1. **Global** — accessible everywhere.
2. **Module** — top-level bindings in an ES module (not global).
3. **Function** — `var` and parameters.
4. **Block** — `let`, `const`, and function declarations inside `{}`.

```js
const global = "global";

function outer() {
  const outerVar = "outer";

  function inner() {
    const innerVar = "inner";
    console.log(global, outerVar, innerVar); // all accessible
  }

  inner();
  // console.log(innerVar); // ReferenceError
}
```

### Scope chain

When a variable is referenced, the engine looks in the current scope, then the outer scope, then the next outer scope, up to the global scope. This chain is determined **lexically** (where the function is written), not dynamically (where it is called).

### Shadowing

```js
const name = "global";

function greet() {
  const name = "local"; // shadows the global
  console.log(name);    // "local"
}
```

> **Avoid:** Global variables. They create hidden coupling and name collisions. Modules and closures keep state contained.

---

## 4. Closures

A **closure** is a function bundled with its lexical environment. It can access variables from its outer scope even after the outer function has returned.

```js
function createCounter() {
  let count = 0;

  return {
    increment() { count++; return count; },
    decrement() { count--; return count; },
    value() { return count; },
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.value();     // 2
```

`count` is private. Nothing outside `createCounter` can modify it directly.

### Why closures matter

- **Data privacy** — the module pattern.
- **Function factories** — pre-configured functions.
- **Event handlers** — remembering state.
- **Debounce/throttle** — remembering timers.
- **Memoization** — caching results.

```js
function once(fn) {
  let called = false;
  let result;

  return function (...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}
```

### The classic loop bug

```js
// Bug: all callbacks log 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// Fix 1: block scope
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}

// Fix 2: closure factory (ES5 style)
for (var j = 0; j < 3; j++) {
  (function (value) {
    setTimeout(() => console.log(value), 0);
  })(j);
}
```

> **Memory note:** Closures keep referenced variables alive. A large object captured by a long-lived handler cannot be garbage-collected until the handler is released.

---

## 5. `this` Binding Rules

`this` is determined by **how a function is called**, not where it is defined (except arrows).

```js
// 1. Global / undefined (strict mode)
function show() { console.log(this); }
show(); // undefined in modules/strict mode

// 2. Method call
const obj = { name: "Ada", show() { console.log(this.name); } };
obj.show(); // "Ada"

// 3. Explicit binding
function greet() { console.log(this.name); }
greet.call({ name: "Grace" });  // "Grace"
greet.apply({ name: "Lin" });   // "Lin"
const bound = greet.bind({ name: "Alan" });
bound();                        // "Alan"

// 4. Constructor call
function Person(name) { this.name = name; }
const ada = new Person("Ada");

// 5. Arrow functions inherit `this`
const team = {
  name: "Core",
  members: ["A", "B"],
  list() {
    return this.members.map((m) => `${this.name}: ${m}`);
  },
};
```

### Lost `this`

```js
const user = {
  name: "Ada",
  greet() { console.log(this.name); },
};

const fn = user.greet;
fn(); // TypeError or undefined — `this` is lost

const safe = user.greet.bind(user);
safe(); // "Ada"
```

> **Gotcha:** Passing `obj.method` as a callback loses its receiver. Bind it or wrap it in an arrow.

---

## 6. `call`, `apply`, `bind`

```js
function introduce(greeting, punctuation) {
  return `${greeting}, ${this.name}${punctuation}`;
}

introduce.call({ name: "Ada" }, "Hello", "!");   // "Hello, Ada!"
introduce.apply({ name: "Ada" }, ["Hi", "."]);  // "Hi, Ada."
const helloAda = introduce.bind({ name: "Ada" });
helloAda("Hey", "!");                           // "Hey, Ada!"
```

| Method | Arguments | Returns |
| :--- | :--- | :--- |
| `call` | Comma-separated | Function result |
| `apply` | Array | Function result |
| `bind` | Comma-separated | New bound function |

---

## 7. Higher-Order Functions

A higher-order function takes or returns another function.

```js
function multiplyBy(factor) {
  return (number) => number * factor;
}

const double = multiplyBy(2);
double(5); // 10

// Composition
const pipe = (...fns) => (value) =>
  fns.reduce((acc, fn) => fn(acc), value);

const process = pipe(
  (s) => s.trim(),
  (s) => s.toLowerCase(),
  (s) => s.replace(/\s+/g, "-")
);

process("  Hello World  "); // "hello-world"
```

### Currying

```js
const curry = (fn) => {
  const arity = fn.length;
  return function curried(...args) {
    return args.length >= arity
      ? fn(...args)
      : (...next) => curried(...args, ...next);
  };
};

const add = curry((a, b, c) => a + b + c);
add(1)(2)(3); // 6
add(1, 2)(3); // 6
```

### Debounce and throttle

```js
function debounce(fn, delay = 300) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

function throttle(fn, limit = 300) {
  let waiting = false;
  return function (...args) {
    if (!waiting) {
      fn.apply(this, args);
      waiting = true;
      setTimeout(() => { waiting = false; }, limit);
    }
  };
}
```

- **Debounce:** run after activity stops (search input).
- **Throttle:** run at most once per interval (scroll handler).

---

## 8. Pure Functions and Side Effects

A **pure function**:

1. Returns the same output for the same input.
2. Has no observable side effects.

```js
// Pure
function add(a, b) { return a + b; }

// Impure: mutates external state
let total = 0;
function addToTotal(n) { total += n; }

// Impure: depends on external state/time
function timestamp() { return Date.now(); }
```

Pure functions are easier to test, reason about, and memoize. Push side effects (DOM, network, storage) to the edges of your application.

---

## ✅ Checkpoint

- [ ] What is the difference between function scope and block scope?
- [ ] Which function forms are hoisted, and which are only usable after their definition?
- [ ] Why can't an arrow function be a constructor, and why shouldn't it be an object method that reads `this`?
- [ ] How does a closure keep a variable alive?
- [ ] Why does `let` fix the `setTimeout` loop bug?
- [ ] What are the five `this` binding rules?
- [ ] When would you use `bind` instead of an arrow function?
- [ ] What is the difference between debounce and throttle?
