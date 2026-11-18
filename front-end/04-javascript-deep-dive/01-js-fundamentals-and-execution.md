# JavaScript Fundamentals & Execution

Before closures, promises, and frameworks, you need a precise model of how JavaScript runs. This file builds that model.

---

## 1. What JavaScript Is

- **High-level** — you do not manage memory manually.
- **Dynamically typed** — types are checked at runtime, not compile time.
- **Single-threaded** — one call stack, one instruction at a time.
- **JIT-compiled** — engines like V8 compile hot code to machine code.
- **Multi-paradigm** — supports procedural, object-oriented, and functional styles.
- **Prototype-based** — objects inherit directly from other objects.

---

## 2. The Runtime

```
┌──────────────────────────── JavaScript Runtime ────────────────────────────┐
│                                                                            │
│  ┌──────────────┐   ┌─────────────────────┐   ┌────────────────────────┐  │
│  │  Call Stack  │   │  Heap (objects)     │   │  Event Loop            │  │
│  │  (LIFO)      │   │                     │   │                        │  │
│  └──────────────┘   └─────────────────────┘   └───────────┬────────────┘  │
│                                                            │               │
│  ┌─────────────────────────────────────────────────────────▼────────────┐ │
│  │ Web APIs: setTimeout, fetch, DOM events, localStorage, ...           │ │
│  └─────────────────────────────────────────────────────────┬────────────┘ │
│                                                            │               │
│  ┌──────────────────────────┐   ┌──────────────────────────▼────────────┐ │
│  │ Microtask queue          │   │ Task (macrotask) queue                │ │
│  │ promises, queueMicrotask │   │ timers, I/O, UI events                │ │
│  └──────────────────────────┘   └───────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────────┘
```

The event loop continuously checks: **is the call stack empty?** If yes, drain all microtasks, then run the next macrotask.

---

## 3. Execution Context

Every time code runs, it does so inside an **execution context**:

- **Global execution context** — created once.
- **Function execution context** — created on every function call.

Each context has:

1. **Variable environment** — `var` bindings, function declarations.
2. **Lexical environment** — `let`/`const` bindings, scope chain.
3. **`this` binding.**
4. **Outer environment reference** — the scope chain.

### Two phases

```js
console.log(a); // undefined (not ReferenceError)
var a = 10;
```

1. **Creation phase:** declarations are registered. `var a` is hoisted and initialized to `undefined`. Function declarations are fully hoisted.
2. **Execution phase:** assignments happen line by line.

```js
console.log(square(4)); // 16 — function declarations are fully hoisted
function square(n) { return n * n; }

console.log(cube(3)); // ReferenceError — function expression is not initialized
const cube = function (n) { return n ** 3; };
```

---

## 4. `var`, `let`, and `const`

```js
var  oldWay = "function-scoped, hoisted to undefined";
let  modern = "block-scoped, hoisted but in TDZ";
const fixed  = "block-scoped, must be initialized, cannot be reassigned";
```

| Feature | `var` | `let` | `const` |
| :--- | :--- | :--- | :--- |
| Scope | Function | Block | Block |
| Hoisted | Yes (`undefined`) | Yes (TDZ) | Yes (TDZ) |
| Redeclare in same scope | Yes | No | No |
| Reassign | Yes | Yes | No |
| Must initialize | No | No | Yes |

### Temporal Dead Zone (TDZ)

```js
{
  // TDZ starts here
  // console.log(name); // ReferenceError: Cannot access 'name' before initialization
  let name = "Ada";
  console.log(name); // "Ada"
}
```

The binding exists but is uninitialized until the declaration is evaluated.

> **Rule:** Use `const` by default. Use `let` only when reassignment is necessary. Avoid `var` in new code.

---

## 5. Primitive Types vs Objects

### Seven primitives

```js
"text"          // string
42              // number (IEEE-754 double)
9007199254740991n // bigint
true            // boolean
undefined       // undefined
null            // null
Symbol("id")    // symbol
```

Everything else is an **object** (arrays, functions, dates, maps, `null` is a historical exception).

```js
typeof "text";        // "string"
typeof 42;            // "number"
typeof 10n;           // "bigint"
typeof true;          // "boolean"
typeof undefined;     // "undefined"
typeof null;          // "object" (historical bug)
typeof Symbol();      // "symbol"
typeof {};            // "object"
typeof [];            // "object"
typeof function(){};  // "function"
```

### Primitives are immutable

```js
let name = "ada";
name.toUpperCase();  // returns "ADA"
console.log(name);   // "ada" — the original string is unchanged
name = name.toUpperCase(); // reassignment, not mutation
```

### Objects are reference values

```js
const a = { count: 1 };
const b = a;      // b points to the same object
b.count = 2;
console.log(a.count); // 2

const c = { ...a }; // shallow copy
c.count = 99;
console.log(a.count); // 2
```

---

## 6. Type Coercion

JavaScript converts types automatically in some operations.

```js
"5" + 1;   // "51"  (string concatenation wins)
"5" - 1;   // 4     (numeric subtraction)
true + 1;  // 2
[] + [];   // ""
[] + {};   // "[object Object]"
1 == "1";  // true  (loose equality coerces)
1 === "1"; // false (strict equality)
```

### Truthy and falsy

Falsy values: `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`.

Everything else is truthy — including `"0"`, `[]`, and `{}`.

```js
if ("0") console.log("truthy"); // runs
if ([]) console.log("truthy");  // runs
```

> **Rule:** Always use `===` and `!==`. Use `Object.is()` for edge cases like `NaN` and `-0`.

```js
NaN === NaN;       // false
Object.is(NaN, NaN); // true
0 === -0;          // true
Object.is(0, -0);  // false
```

---

## 7. Numbers and Precision

```js
0.1 + 0.2;                 // 0.30000000000000004
Number.isInteger(5);       // true
Number.parseFloat("3.14"); // 3.14
Number.parseInt("42px");   // 42
(1234.5678).toFixed(2);    // "1234.57"

// Safe integer range
Number.MAX_SAFE_INTEGER; // 9007199254740991
9007199254740992 === 9007199254740993; // true — precision lost
```

Use `BigInt` for integers beyond the safe range:

```js
const big = 9007199254740993n;
big + 1n; // 9007199254740994n
```

> **Gotcha:** Never compare floating-point currency directly. Store cents as integers or use a decimal library.

---

## 8. Control Flow Essentials

Control flow decides **which lines run, in what order, and how many times**. Every example below is self-contained, so you can paste it into `app.js` or the DevTools console and run it as-is.

### 8.1 `if` / `else if` / `else`

```js
function gradeFor(score) {
  if (score >= 90) {
    return "A";
  } else if (score >= 80) {
    return "B";
  } else if (score >= 70) {
    return "C";
  } else {
    return "F";
  }
}

console.log(gradeFor(95)); // "A"
console.log(gradeFor(85)); // "B"
console.log(gradeFor(60)); // "F"
```

Branches are tested **top to bottom**, and the first truthy branch wins. That is why order matters: if you tested `score >= 70` first, every score of 70 or more would return `"C"`.

- The condition uses **truthiness** (see §6): `0`, `""`, `null`, `undefined`, `NaN`, and `false` are falsy; everything else is truthy.
- Braces `{ }` are optional for a single statement, but always write them — it prevents the classic "dangling `else`" bug.
- `if (x = 5)` is valid JavaScript: it assigns, then tests. Use `===` unless assignment is intended.

### 8.2 Conditional (ternary) operator

`condition ? whenTrue : whenFalse` is an **expression**, so it can be returned or assigned.

```js
function itemLabel(count) {
  return count === 1 ? "item" : "items";
}

console.log(itemLabel(1)); // "item"
console.log(itemLabel(4)); // "items"

const status = "success";
const icon = status === "success" ? "✅" : status === "error" ? "❌" : "⏳";
console.log(icon); // "✅"
```

> **Rule:** Use a ternary for a single either/or value. For two or more conditions, use `if`/`switch` or a lookup object — nested ternaries are hard to read and easy to mis-order.

```js
// Cleaner than a nested ternary:
const icons = { success: "✅", error: "❌", loading: "⏳" };

function iconFor(status) {
  return icons[status] ?? "❓";
}

console.log(iconFor("error"));  // "❌"
console.log(iconFor("unknown")); // "❓"
```

### 8.3 `switch`

`switch` compares the expression against each `case` using **strict equality (`===`)**, then runs every statement until it hits `break` or `return`.

```js
function renderStatus(status) {
  switch (status) {
    case "loading":
      return "⏳ Loading…";
    case "error":
      return "❌ Something went wrong";
    case "empty":
    case "success": // grouped cases share one branch
      return "✅ Content ready";
    default:
      return "❓ Unknown status";
  }
}

console.log(renderStatus("loading")); // "⏳ Loading…"
console.log(renderStatus("empty"));   // "✅ Content ready"
console.log(renderStatus("nope"));    // "❓ Unknown status"
```

Watch out for **fall-through** — forgetting `break` makes the next case run too:

```js
function describe(value) {
  let result = "";
  switch (value) {
    case "a":
      result += "a";
    // missing break: execution falls into "b"
    case "b":
      result += "b";
  }
  return result;
}

console.log(describe("a")); // "ab" — probably not what you wanted
console.log(describe("b")); // "b"
```

Notes:

- `switch` uses `===`, so `case "1"` does **not** match the number `1`.
- `default` can go anywhere, but keep it last for readability.
- If every case returns, you do not need `break`.
- For many string → value mappings, a lookup object (`8.2`) is usually simpler than `switch`.

### 8.4 Guard clauses (early return)

Deeply nested `if`s are hard to follow. Invert the checks and return early:

```js
// Harder to read
function priceAfterDiscountNested(price, user) {
  if (typeof price === "number" && !Number.isNaN(price)) {
    if (price > 0) {
      if (user && user.isMember) {
        return price * 0.9;
      } else {
        return price;
      }
    }
  }
  return null;
}

// Easier to read: fail fast, then handle the happy path
function priceAfterDiscount(price, user) {
  if (typeof price !== "number" || Number.isNaN(price)) return null;
  if (price <= 0) return 0;
  if (!user?.isMember) return price;
  return price * 0.9;
}

console.log(priceAfterDiscount(100, { isMember: true }));  // 90
console.log(priceAfterDiscount(100, { isMember: false })); // 100
console.log(priceAfterDiscount("100", {}));                // null
```

> **Rule:** Validate inputs at the top, return early, then keep the main logic at one indentation level.

### 8.5 Nullish coalescing `??`

`??` falls back **only** when the left side is `null` or `undefined`. `||` falls back on **every falsy** value.

| Value on the left | `value \|\| fallback` | `value ?? fallback` |
| :--- | :--- | :--- |
| `null` | fallback | fallback |
| `undefined` | fallback | fallback |
| `0` | fallback ❌ | `0` ✅ |
| `""` | fallback ❌ | `""` ✅ |
| `false` | fallback ❌ | `false` ✅ |
| `"text"` | `"text"` | `"text"` |

```js
function pointsLabel(points) {
  const safe = points || 10; // BUG: a real score of 0 becomes 10
  return `${safe} points`;
}

function fixedPointsLabel(points) {
  const safe = points ?? 10; // only null/undefined fall back
  return `${safe} points`;
}

console.log(pointsLabel(0));      // "10 points" — wrong
console.log(fixedPointsLabel(0)); // "0 points" — correct
console.log(fixedPointsLabel());  // "10 points"
```

> **Gotcha:** `??` cannot be mixed with `||` or `&&` without parentheses — `a ?? b || c` is a `SyntaxError`. Write `(a ?? b) || c` or rethink the logic.

### 8.6 Optional chaining `?.`

`?.` stops evaluating and returns `undefined` when the value before it is `null` or `undefined`, instead of throwing.

```js
const user = {
  name: "Ada",
  address: { city: "London" },
  greet() {
    return `Hi, ${this.name}`;
  },
};

const guest = { name: "Guest" };

console.log(user.address?.city);  // "London"
console.log(guest.address?.city); // undefined — no TypeError
console.log(user.greet?.());      // "Hi, Ada"
console.log(guest.greet?.());     // undefined — method missing
console.log(user.tags?.[0]);      // undefined — array missing
```

Important details:

- `?.` short-circuits the **rest of the chain**: in `a?.b.c`, if `a` is nullish, `.c` is never evaluated.
- It does **not** protect against undeclared variables: `unknownVar?.x` still throws `ReferenceError`.
- Do not wrap everything in `?.`. If `null` means a bug, let it throw so you notice. Use `?.` for data that is genuinely optional.

### 8.7 Logical assignment operators

These combine a logical check with assignment.

```js
function applyDefaults(settings) {
  settings.theme ||= "light"; // assign if current value is falsy
  settings.retries ??= 3;     // assign only if null/undefined
  settings.debug &&= false;   // assign only if current value is truthy
  return settings;
}

console.log(applyDefaults({ theme: "", retries: 0, debug: true }));
// { theme: "light", retries: 0, debug: false }

console.log(applyDefaults({ theme: "dark", retries: null }));
// { theme: "dark", retries: 3 }
```

### 8.8 Which one should I use?

| Situation | Use |
| :--- | :--- |
| Validate and exit early | `if` + `return` |
| One of two values | Ternary `? :` |
| Match one of many fixed values | `switch` or a lookup object |
| Default for missing values | `??` |
| Default that should also replace `0` / `""` | `\|\|` (rare, be intentional) |
| Possibly missing nested data | `?.` |
| Set a value only when absent | `??=`, `\|\|=`, `&&=` |

---

## 9. Loops

A loop repeats a block until a condition changes. Choose the loop that matches the shape of your data — picking the wrong one is where most loop bugs come from.

### 9.1 Classic `for`

Use when you need a counter, an index, or custom steps.

```js
function sumTo(n) {
  let total = 0;
  for (let i = 1; i <= n; i++) {
    total += i;
  }
  return total;
}

console.log(sumTo(4)); // 10  (1+2+3+4)

// Reverse and custom step
const letters = ["a", "b", "c", "d"];
for (let i = letters.length - 1; i >= 0; i -= 2) {
  console.log(letters[i]); // "d", "b"
}
```

Anatomy: `for (initialization; condition; update)`. The condition is checked **before** every iteration, so if it is false at the start the body never runs. Each `let` creates a fresh binding per iteration (important for closures — see §9.8).

### 9.2 `while` and `do...while`

Use `while` when you do not know the number of iterations in advance (retry loops, reading until empty, polling).

```js
function countdown(start) {
  let current = start;
  while (current > 0) {
    console.log(current);
    current--; // without this the loop never ends
  }
  console.log("Liftoff!");
}

countdown(3); // 3, 2, 1, Liftoff!
```

`do...while` checks the condition **after** the body, so the body always runs at least once:

```js
function askUntilYes(answers) {
  let index = 0;
  let answer;
  do {
    answer = answers[index++];
    console.log(`User answered: ${answer}`);
  } while (answer !== "yes" && index < answers.length);
  return answer;
}

askUntilYes(["nope", "nope", "yes"]);
// User answered: nope
// User answered: nope
// User answered: yes
```

> **Gotcha:** Infinite loops freeze the single thread. Make sure something in the body moves the condition toward `false`, and give retry loops a maximum attempt count.

### 9.3 `for...of` — iterate values

Works with any **iterable**: arrays, strings, `Map`, `Set`, `arguments`, generators.

```js
const fruits = ["apple", "banana", "cherry"];

for (const fruit of fruits) {
  console.log(fruit); // "apple", "banana", "cherry"
}

// Need the index? Use .entries()
for (const [index, fruit] of fruits.entries()) {
  console.log(index, fruit); // 0 "apple", 1 "banana", ...
}

// Strings iterate by character
for (const char of "hi") {
  console.log(char); // "h", "i"
}

// Maps iterate as [key, value] pairs
const prices = new Map([
  ["apple", 1],
  ["banana", 2],
]);

for (const [name, price] of prices) {
  console.log(`${name}: $${price}`);
}
```

`for...of` reads **values**, supports `break`/`continue`, and cannot be used on plain objects (they are not iterable — use `Object.entries()`).

### 9.4 `for...in` — iterate keys

`for...in` walks enumerable **property keys** (as strings), including **inherited** ones.

```js
const config = { theme: "dark", lang: "en" };

for (const key in config) {
  console.log(key, config[key]); // "theme dark", "lang en"
}

// Inherited properties are included too
const base = { inherited: true };
const child = Object.create(base);
child.own = true;

for (const key in child) {
  console.log(key); // "own", then "inherited"
}

// Guard with Object.hasOwn when only own keys matter
for (const key in child) {
  if (Object.hasOwn(child, key)) {
    console.log(key); // "own" only
  }
}
```

- Prefer `Object.keys()` / `Object.entries()` over `for...in` — they return own keys only and behave more predictably.
- **Never use `for...in` on arrays.** It yields index strings (`"0"`, `"1"`), can include extra properties, and does not guarantee order for non-index keys.

```js
// Avoid
const arr = ["a", "b"];
for (const i in arr) console.log(typeof i, i); // "string" "0", "string" "1"

// Prefer
for (const value of arr) console.log(value);    // "a", "b"
for (const [i, value] of arr.entries()) console.log(i, value); // 0 "a", 1 "b"
```

### 9.5 Loops vs array methods

Array methods express *what* you want instead of *how* to loop, and they return new values.

| Method | Returns | Use it to |
| :--- | :--- | :--- |
| `forEach` | `undefined` | Run a side effect for each item |
| `map` | New array (same length) | Transform every item |
| `filter` | New array (≤ length) | Keep items that pass a test |
| `reduce` | Any single value | Sum, group, build an object |
| `find` | First match or `undefined` | Get one item |
| `findIndex` | Index or `-1` | Locate one item |
| `some` | Boolean | Ask "does at least one pass?" |
| `every` | Boolean | Ask "do all pass?" |

```js
const numbers = [1, 2, 3, 4, 5];

console.log(numbers.map((n) => n * 2));            // [2, 4, 6, 8, 10]
console.log(numbers.filter((n) => n % 2 === 1));   // [1, 3, 5]
console.log(numbers.reduce((sum, n) => sum + n, 0)); // 15
console.log(numbers.find((n) => n > 3));           // 4
console.log(numbers.findIndex((n) => n > 3));      // 3
console.log(numbers.some((n) => n > 4));           // true
console.log(numbers.every((n) => n > 0));          // true

numbers.forEach((n, index) => console.log(index, n));
```

Key differences:

- `map`/`filter`/`reduce` **return** a result; `forEach` always returns `undefined`.
- `forEach` cannot be stopped with `break`. Use `for...of`, or `some`/`every`/`find` when you need early exit.
- `filter` + `map` is fine for small arrays. For very large arrays, a single `for...of` loop avoids creating intermediate arrays.

```js
// Early exit with for...of
function findFirstNegative(values) {
  for (const value of values) {
    if (value < 0) return value;
  }
  return null;
}

console.log(findFirstNegative([3, -2, -9])); // -2
```

### 9.6 `break`, `continue`, and labels

- `break` exits the **innermost** loop immediately.
- `continue` skips the rest of the current iteration and moves to the next one.

```js
for (const n of [1, 2, 3, 4, 5]) {
  if (n === 2) continue; // skip 2
  if (n === 5) break;    // stop before 5
  console.log(n);        // 1, 3, 4
}
```

A **label** lets `break`/`continue` target an outer loop:

```js
function findPair(grid) {
  outer: for (let row = 0; row < grid.length; row++) {
    for (let col = 0; col < grid[row].length; col++) {
      if (grid[row][col] === "X") {
        console.log(`Found X at row ${row}, col ${col}`);
        break outer; // exits both loops
      }
    }
  }
}

findPair([
  ["O", "O"],
  ["O", "X"],
]); // "Found X at row 1, col 1"
```

Labels are rarely needed. If you reach for one, consider extracting the inner loop into a function and using `return` instead — it is clearer.

### 9.7 Which loop should I use?

| Goal | Use |
| :--- | :--- |
| Repeat a fixed number of times / need an index | `for` |
| Repeat until a condition changes | `while` |
| Must run at least once | `do...while` |
| Iterate array/string/`Map`/`Set` values | `for...of` |
| Iterate object keys (own) | `Object.keys()` / `Object.entries()` + `for...of` |
| Transform / filter / sum | `map` / `filter` / `reduce` |
| Stop when a match is found | `for...of` + `return`, or `some` / `find` |
| Run something for each item, no result | `forEach` (or `for...of`) |

### 9.8 Loop pitfalls

**1. `var` in a loop with an async callback.** All callbacks share one binding:

```js
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var:", i), 0);
}
// var: 3, var: 3, var: 3 — the loop finished before the timers ran

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let:", j), 0);
}
// let: 0, let: 1, let: 2 — each iteration gets its own binding
```

(`setTimeout` and the event loop are covered in `06-async-javascript-event-loop.md`.)

**2. Mutating an array while iterating over it.** Indexes shift and items get skipped:

```js
const items = [1, 2, 4, 5];

items.forEach((item, index) => {
  if (item % 2 === 0) items.splice(index, 1);
});

console.log(items); // [1, 4, 5] — 4 was skipped
```

Use `filter` instead, or iterate over a copy:

```js
const original = [1, 2, 4, 5];
const odds = original.filter((n) => n % 2 !== 0);
console.log(odds);     // [1, 5]
console.log(original); // [1, 2, 4, 5] — untouched
```

**3. Off-by-one errors.** `i <= array.length` reads one past the end:

```js
const list = ["a", "b", "c"];

for (let k = 0; k < list.length; k++) {
  console.log(list[k]); // "a", "b", "c"
}

console.log(list[list.length]); // undefined — the classic mistake
```

**4. Modifying a `const` in a loop body.** `const` prevents reassignment of the *binding*, not mutation of the object:

```js
const totals = [0, 0];
for (let k = 0; k < 2; k++) {
  totals[k] += 1; // fine: mutating the array, not rebinding `totals`
}
console.log(totals); // [1, 1]
```

---

## 10. Error Handling

Errors are normal. Network requests fail, JSON is malformed, and users type things you did not expect. Good error handling means **failing loudly where you cannot recover** and **recovering deliberately where you can**.

### 10.1 `try` / `catch` / `finally`

```js
function parseConfig(text) {
  try {
    const config = JSON.parse(text);
    return config;
  } catch (error) {
    console.error(`Invalid config: ${error.message}`);
    return null;
  } finally {
    console.log("parseConfig finished");
  }
}

console.log(parseConfig('{"debug":true}'));
// { debug: true }

console.log(parseConfig("{oops"));
// Invalid config: Expected property name or '}' in JSON at position 1
// null
// (exact JSON message varies by engine)
```

How the blocks run:

1. `try` — code that might throw.
2. `catch (error)` — runs **only** if something inside `try` threw. The thrown value arrives as `error`.
3. `finally` — runs **always**: after success, after an error, and even after `return` or `throw`.

```js
function trap() {
  try {
    return "try";
  } finally {
    console.log("finally always runs");
  }
}

function overridden() {
  try {
    return "try";
  } finally {
    return "finally"; // a return in finally discards the previous return
  }
}

console.log(trap());       // "finally always runs", then "try"
console.log(overridden()); // "finally"
```

> **Rule:** Use `finally` to release resources (close a connection, clear a spinner, remove a listener). Do not put `return` inside `finally`.

### 10.2 How errors propagate

When a function throws and does not catch, the error travels **up the call stack** until some caller catches it. If nobody catches it, the script (or the current task) stops and the browser reports an *uncaught* error.

```js
function loadUser(id) {
  if (!id) {
    throw new TypeError("id is required");
  }
  return { id };
}

function loadProfile(id) {
  return { user: loadUser(id) }; // does not catch — lets the error bubble
}

function boot(id) {
  try {
    return loadProfile(id);
  } catch (error) {
    console.error(`${error.name}: ${error.message}`);
    return null;
  }
}

console.log(boot(7)); // { user: { id: 7 } }
console.log(boot());  // "TypeError: id is required", then null
```

The throw inside `loadUser` skips every remaining line of `loadUser`, `loadProfile`, and `try` in `boot`, landing directly in `boot`'s `catch`.

### 10.3 Throwing errors

Use `throw` to signal that a function cannot do its job. **Throw `Error` objects**, never strings or plain objects — only error objects give you `name`, `message`, `stack`, and `cause`.

```js
function divide(a, b) {
  if (typeof a !== "number" || typeof b !== "number") {
    throw new TypeError("divide expects two numbers");
  }
  if (b === 0) {
    throw new RangeError("Cannot divide by zero");
  }
  return a / b;
}

try {
  console.log(divide(10, 2)); // 5
  console.log(divide(1, 0));  // throws — jumps straight to catch
  console.log("never runs");
} catch (error) {
  console.log(`${error.name}: ${error.message}`); // "RangeError: Cannot divide by zero"
} finally {
  console.log("done");
}
```

`throw` vs `return`:

| | `return` | `throw` |
| :--- | :--- | :--- |
| Caller can ignore it | Yes | No (must catch or crash) |
| Skips the rest of the function | Yes | Yes |
| Carries a stack trace | No | Yes |
| Meant for | Normal results | Exceptional situations |

> **Rule:** Validate at the boundary (user input, API responses) and throw there. Internal functions can then assume valid data, which keeps them simple.

### 10.4 Anatomy of an error object

```js
const error = new Error("Something failed");

console.log(error.name);    // "Error"
console.log(error.message); // "Something failed"
console.log(error.stack);   // multi-line stack trace (format is engine-specific)
```

- `name` — the error type (`"TypeError"`, `"RangeError"`, …).
- `message` — human-readable description. Include **what** failed and the offending value.
- `stack` — where it happened. Do not show this to users; keep it in logs.
- `cause` (ES2022) — the original error, when you wrap one error in another.

```js
function readSettings(raw) {
  try {
    return JSON.parse(raw);
  } catch (error) {
    // Keep the original error attached for debugging
    throw new Error("Settings are not valid JSON", { cause: error });
  }
}

try {
  readSettings("{nope");
} catch (error) {
  console.log(error.message);    // "Settings are not valid JSON"
  console.log(error.cause.name); // "SyntaxError" — the real cause
}
```

### 10.5 Built-in error types

| Error | Typical cause | Example that throws it |
| :--- | :--- | :--- |
| `ReferenceError` | Using an undeclared variable | `console.log(notDeclared)` |
| `TypeError` | Wrong type / calling a non-function / property of `null` | `null.foo`, `undefined()` |
| `SyntaxError` | Invalid code at parse time | `JSON.parse("{oops")` |
| `RangeError` | Invalid number range / stack overflow | `new Array(-1)` |
| `URIError` | Malformed URI passed to a URI function | `decodeURIComponent("%")` |
| `DOMException` | Browser API failure | `localStorage` in a private context, aborted `fetch` |

```js
function tryIt(fn) {
  try {
    fn();
    console.log("no error");
  } catch (error) {
    console.log(`${error.name}: ${error.message}`);
  }
}

tryIt(() => console.log(notDeclared));   // ReferenceError
tryIt(() => null.foo);                   // TypeError
tryIt(() => JSON.parse("{oops"));        // SyntaxError
tryIt(() => new Array(-1));              // RangeError
tryIt(() => decodeURIComponent("%"));    // URIError
```

> **Note:** A `SyntaxError` in your own source file happens **before** any code runs, so `try/catch` around it cannot help — fix the file. You *can* catch syntax errors raised at runtime by parsers such as `JSON.parse`.

### 10.6 Catch precisely, then rethrow what you cannot handle

A `catch` block should handle the errors it understands and let everything else continue. Otherwise you hide real bugs.

```js
function validateAge(age) {
  if (!Number.isInteger(age)) {
    throw new TypeError("Age must be an integer");
  }
  if (age < 0) {
    throw new RangeError("Age cannot be negative");
  }
  return age;
}

function submitAge(age) {
  try {
    return `Saved age ${validateAge(age)}`;
  } catch (error) {
    if (error instanceof TypeError) return "Please enter a whole number.";
    if (error instanceof RangeError) return "Age must be 0 or more.";
    throw error; // unexpected error — do not swallow it
  }
}

console.log(submitAge(30));   // "Saved age 30"
console.log(submitAge("30")); // "Please enter a whole number."
console.log(submitAge(-1));   // "Age must be 0 or more."
```

Use `instanceof` to branch on the error class. In cross-realm situations (iframes, workers) `instanceof` can fail; compare `error.name` as a fallback.

### 10.7 Custom error classes

Create a subclass when callers need to distinguish *your* failures from built-ins.

```js
class ValidationError extends Error {
  constructor(field, message) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
  }
}

function requireName(user) {
  if (!user.name) {
    throw new ValidationError("name", "Name is required");
  }
  return user.name;
}

try {
  requireName({});
} catch (error) {
  if (error instanceof ValidationError) {
    console.log(`${error.field}: ${error.message}`); // "name: Name is required"
  } else {
    throw error;
  }
}
```

> **Why subclass?** `error instanceof ValidationError` lets a form handler show field-level messages while a generic logger still catches everything else.

### 10.8 Async errors (preview)

`try/catch` works with `await` inside `async` functions. It does **not** catch a promise you forgot to `await`.

```js
async function loadData(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error("loadData failed:", error.message);
    return null;
  }
}

loadData("https://example.com/api/data");
```

- Throwing inside an `async` function **rejects** its returned promise.
- A rejected promise that nobody handles becomes an `unhandledrejection` event.
- Full details are in `06-async-javascript-event-loop.md`.

### 10.9 Error-handling rules

1. **Throw error objects**, not strings: `throw new Error("msg")`.
2. **Fail fast** on invalid input, at the boundary of your code.
3. **Catch only what you can handle.** Otherwise, rethrow.
4. **Never swallow silently.** An empty `catch {}` makes bugs invisible. At minimum, log with context.
5. **Do not use exceptions for normal control flow** — return a value for expected outcomes.
6. **Clean up in `finally`**, or close resources as soon as they are no longer needed.
7. **Never show `stack` to users**; show a friendly `message` and log the details.
8. **Give context in messages**: `"Failed to load user 42: HTTP 500"` beats `"Error"`.
9. **`console.error` is not handling** — decide whether to recover, retry, or surface the error to the UI.

---

## ✅ Checkpoint

- [ ] What are the two phases of an execution context?
- [ ] Why does `console.log(a)` print `undefined` before `var a = 1`?
- [ ] What is the Temporal Dead Zone?
- [ ] Why is `typeof null === "object"`?
- [ ] When should you use `??` instead of `||`?
- [ ] Why is `0.1 + 0.2 !== 0.3`?
- [ ] Why does a `switch` case need `break` or `return`?
- [ ] What is the difference between `for...of` and `for...in`?
- [ ] Why does `var` in a loop log the final value in `setTimeout`, while `let` logs each iteration?
- [ ] Why does a mutation inside `forEach` skip items?
- [ ] What does `finally` do when the `try` block returns?
- [ ] When should a `catch` block rethrow instead of returning a fallback?
