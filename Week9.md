# Modern JavaScript — A Deep, Example-Driven Guide

> Goal: understand *why* each feature exists, not just its syntax. Every section builds on the previous one, so read in order the first time. All examples run in Node 20+ or any modern browser console.

**Topics covered**

1. `let` / `const` (and why `var` is out)
2. Types and type coercion
3. Arrow functions
4. Callbacks
5. Objects
6. Destructuring
7. Spread and rest
8. Array methods
9. Closures
10. ES modules
11. Putting it together

---

## 1. `let`, `const`, and the end of `var`

### 1.1 The problem with `var`

Before ES2015, `var` was the only way to declare a variable. It has two behaviours that cause bugs:

1. **Function scope, not block scope.** A `var` inside an `if` or `for` leaks out.
2. **Hoisting with `undefined`.** The declaration is moved to the top of its scope and initialised to `undefined`, so you can read it before the line that declares it.

```js
function demoVar() {
  if (true) {
    var secret = "leaked";
  }
  console.log(secret); // "leaked" — the block did not contain it
}
demoVar();

console.log(early); // undefined (no error!)
var early = 5;
```

The second one is nasty: the code "works" but silently uses `undefined`.

### 1.2 `let` — a variable that can be reassigned

`let` is **block-scoped**. A block is anything between `{ }`: an `if`, a `for`, a plain block, a function body.

```js
let count = 0;
count = count + 1; // OK, reassignment allowed
console.log(count); // 1

if (true) {
  let inner = "only here";
}
// console.log(inner); // ReferenceError: inner is not defined
```

You cannot redeclare the same name twice in the same scope:

```js
let city = "Hyderabad";
// let city = "Bengaluru"; // SyntaxError: Identifier 'city' has already been declared
city = "Bengaluru"; // reassignment is fine
```

### 1.3 `const` — a binding that cannot be reassigned

`const` is also block-scoped, but the **binding** cannot be reassigned. It must be initialised at declaration.

```js
const PI = 3.14159;
// PI = 3; // TypeError: Assignment to constant variable.

// const x;  // SyntaxError: Missing initializer in const declaration
```

**Critical misconception:** `const` does *not* make the value immutable. It only stops you re-pointing the name.

```js
const user = { name: "Deepak" };
user.name = "Chand";      // allowed — we mutated the object
console.log(user);        // { name: 'Chand' }

// user = { name: "x" };  // TypeError — can't re-point the binding

const nums = [1, 2, 3];
nums.push(4);             // allowed
console.log(nums);        // [1, 2, 3, 4]
```

If you truly want a shallow-frozen object, use `Object.freeze`:

```js
const config = Object.freeze({ debug: false });
config.debug = true;      // silently ignored (throws in strict mode)
console.log(config.debug); // false
```

### 1.4 The temporal dead zone (TDZ)

`let` and `const` are hoisted too, but unlike `var` they are **not initialised** until execution reaches the declaration. The span from the start of the block to the declaration line is the *temporal dead zone*. "Temporal" because it depends on execution order (time), not where the code sits on the page.

```js
{
  // TDZ for `age` starts here
  // console.log(age); // ReferenceError: Cannot access 'age' before initialization
  let age = 30;        // TDZ ends here
  console.log(age);    // 30
}
```

A function can *mention* a TDZ variable safely, as long as it is *called* after the declaration:

```js
{
  const show = () => console.log(value); // fine to define here
  let value = 3;
  show(); // 3 — called outside the TDZ
}
```

**Why this is a feature:** a ReferenceError at the exact bad line is far easier to debug than a silent `undefined`.

### 1.5 The classic loop bug that `let` fixes

```js
// var: one shared variable across all iterations
const fnsVar = [];
for (var i = 0; i < 3; i++) {
  fnsVar.push(() => i);
}
console.log(fnsVar.map((f) => f())); // [3, 3, 3]

// let: a fresh binding per iteration
const fnsLet = [];
for (let j = 0; j < 3; j++) {
  fnsLet.push(() => j);
}
console.log(fnsLet.map((f) => f())); // [0, 1, 2]
```

This is closures meeting block scope (we return to this in section 9).

### 1.6 Rules of thumb

- Default to **`const`**.
- Use **`let`** only when you genuinely reassign (counters, accumulators, state that changes).
- Never use **`var`** in new code.

Reasoning: `const` tells the reader "this name always points at the same thing," which removes a whole category of "who changed this?" questions.

---

## 2. Types

JavaScript is **dynamically typed**: variables have no type, *values* do. A variable can hold a number now and a string later.

### 2.1 The primitives

There are seven primitive types. Primitives are immutable and compared by value.

| Type | Example | Notes |
|---|---|---|
| `string` | `"hi"`, `'hi'`, `` `hi` `` | UTF-16 text |
| `number` | `42`, `3.14`, `NaN`, `Infinity` | 64-bit float, one type for ints and floats |
| `bigint` | `123n` | arbitrary-size integers |
| `boolean` | `true`, `false` | |
| `undefined` | `undefined` | "no value assigned" |
| `null` | `null` | "intentionally empty" |
| `symbol` | `Symbol("id")` | unique identifiers |

Everything else is an **object** (arrays, functions, dates, maps, plain objects).

```js
console.log(typeof "hi");        // "string"
console.log(typeof 42);          // "number"
console.log(typeof 10n);         // "bigint"
console.log(typeof true);        // "boolean"
console.log(typeof undefined);   // "undefined"
console.log(typeof Symbol("x")); // "symbol"
console.log(typeof {});          // "object"
console.log(typeof []);          // "object"   <- arrays are objects
console.log(typeof (() => {}));  // "function"
console.log(typeof null);        // "object"   <- a famous historical bug
```

Checking for arrays and null properly:

```js
console.log(Array.isArray([]));  // true
console.log(null === null);      // true — to test null, compare directly
```

### 2.2 Numbers and their quirks

All numbers are IEEE-754 doubles, so floating-point error shows up:

```js
console.log(0.1 + 0.2);                 // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3);         // false
console.log(Math.abs(0.1 + 0.2 - 0.3) < Number.EPSILON); // true — safe comparison

console.log(Number.MAX_SAFE_INTEGER);   // 9007199254740991
console.log(9007199254740991 + 2);      // 9007199254740992 (precision lost)
console.log(9007199254740991n + 2n);    // 9007199254740993n (BigInt is exact)
```

`NaN` ("not a number") is a number that is not equal to itself:

```js
console.log(Number("abc"));        // NaN
console.log(NaN === NaN);          // false
console.log(Number.isNaN(NaN));    // true — use this
console.log(Number.isNaN("abc"));  // false (does not coerce)
console.log(isNaN("abc"));         // true  (old global one coerces — avoid)
```

### 2.3 Strings and template literals

Template literals (backticks) allow interpolation and multi-line text:

```js
const name = "Deepak";
const years = 1;
console.log(`${name} has ${years} year${years === 1 ? "" : "s"} of experience`);

const multi = `line one
line two`;
console.log(multi);
```

Useful string methods:

```js
const s = "  Modern JavaScript  ";
console.log(s.trim());                     // "Modern JavaScript"
console.log(s.trim().toUpperCase());       // "MODERN JAVASCRIPT"
console.log(s.includes("Java"));           // true
console.log("a,b,c".split(","));           // ["a", "b", "c"]
console.log("abc".padStart(5, "*"));       // "**abc"
console.log("x".repeat(3));                // "xxx"
console.log("hello world".replaceAll("o", "0")); // "hell0 w0rld"
console.log("hello".at(-1));               // "o"
```

Strings are immutable. Methods return new strings:

```js
const word = "cat";
word.toUpperCase();
console.log(word); // still "cat"
```

### 2.4 `null` vs `undefined`

- `undefined`: the engine's default "nothing here yet" (uninitialised variable, missing property, function with no return).
- `null`: a value *you* set to say "deliberately empty".

```js
let a;
console.log(a);                 // undefined
const obj = {};
console.log(obj.missing);       // undefined
function noReturn() {}
console.log(noReturn());        // undefined

let selectedUser = null;        // "no user selected yet", on purpose
```

### 2.5 Truthy and falsy

In conditions, values are coerced to booleans. Exactly these are **falsy**:

`false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`

Everything else is truthy — including `"0"`, `"false"`, `[]`, and `{}`.

```js
console.log(Boolean(""));       // false
console.log(Boolean("0"));      // true
console.log(Boolean([]));       // true   <- empty array is truthy
console.log(Boolean({}));       // true
```

### 2.6 `==` vs `===`

`===` (strict) compares without type conversion. `==` (loose) coerces first, with surprising rules.

```js
console.log(1 === "1");   // false
console.log(1 == "1");    // true
console.log(0 == "");     // true
console.log(null == undefined);  // true
console.log(null === undefined); // false
console.log([] == false); // true  (!)
```

**Rule:** always use `===` and `!==`. The one accepted exception is `x == null`, which is true for both `null` and `undefined`.

### 2.7 Short-circuiting and the nullish operators

```js
// || returns the first truthy value
console.log("" || "default");     // "default"
console.log(0 || 10);             // 10  <- 0 is falsy, so we lose a valid 0

// ?? returns the right side only for null/undefined
console.log(0 ?? 10);             // 0   <- keeps the valid 0
console.log(null ?? 10);          // 10
console.log(undefined ?? "n/a");  // "n/a"

// Optional chaining ?. stops at null/undefined instead of throwing
const user = { profile: { city: "Hyderabad" } };
console.log(user.profile?.city);       // "Hyderabad"
console.log(user.address?.street);     // undefined (no TypeError)
console.log(user.getName?.());         // undefined (method may not exist)

// Logical assignment
let theme;
theme ??= "dark";   // assign only if null/undefined
console.log(theme); // "dark"
```

**Why `??` matters:** `||` treats `0`, `""`, and `false` as "missing." For things like `quantity`, `price`, or `volume`, that is a bug. `??` only treats real absence as missing.

### 2.8 Type conversion cheat sheet

```js
console.log(Number("42"));      // 42
console.log(Number(""));        // 0
console.log(Number("4px"));     // NaN
console.log(parseInt("4px"));   // 4
console.log(parseFloat("3.5kg")); // 3.5
console.log(String(123));       // "123"
console.log((255).toString(16)); // "ff"
console.log(Boolean("hi"));     // true
console.log(+"7");              // 7  (unary plus)
console.log("5" + 1);           // "51"  (+ with a string concatenates)
console.log("5" - 1);           // 4     (- always converts to number)
```

---

## 3. Arrow functions

### 3.1 Three ways to write a function

```js
// 1. Declaration — hoisted completely, can be called before it appears
console.log(add(2, 3)); // 5
function add(a, b) {
  return a + b;
}

// 2. Expression — only exists after the line runs
const mul = function (a, b) {
  return a * b;
};

// 3. Arrow function
const sub = (a, b) => a - b;
console.log(mul(2, 3), sub(5, 2)); // 6 3
```

### 3.2 Arrow syntax variations

```js
const square = (x) => x * x;          // one param, implicit return
const squareB = x => x * x;           // parens optional for exactly one param
const noArgs = () => 42;              // zero params need parens
const sum = (a, b) => a + b;          // multiple params

const withBody = (a, b) => {          // braces => you must write `return`
  const total = a + b;
  return total * 2;
};

// Returning an object literal: wrap in parentheses
const makeUser = (name) => ({ name, active: true });
console.log(makeUser("Deepak")); // { name: 'Deepak', active: true }

// Without the parens, { } is parsed as a function body, not an object:
const broken = (name) => { name };   // just an expression statement, no return
console.log(broken("x"));            // undefined
// (a fuller literal like `{ name, active: true }` here is a SyntaxError)
```

### 3.3 The big difference: `this`

Regular functions get their own `this`, decided by **how they are called**. Arrow functions have **no `this` of their own**; they use the `this` of the surrounding code (lexical `this`).

```js
const timer = {
  seconds: 0,
  startBroken() {
    // regular function inside: `this` is NOT the timer
    return [1, 2, 3].map(function () {
      return this === timer; // false (undefined or global depending on mode)
    });
  },
  startFixed() {
    // arrow function: inherits `this` from startFixed
    return [1, 2, 3].map(() => this === timer);
  },
};
console.log(timer.startBroken()); // [false, false, false]
console.log(timer.startFixed());  // [true, true, true]
```

Practical case — a counter object:

```js
const counter = {
  count: 0,
  incrementLater() {
    setTimeout(() => {
      this.count++;           // `this` is `counter` thanks to the arrow
      console.log(this.count); // 1
    }, 0);
  },
};
counter.incrementLater();
```

### 3.4 When NOT to use arrows

1. **Object methods** — you usually want `this` to be the object:

```js
const person = {
  name: "Asha",
  greetArrow: () => `Hi ${this?.name}`,   // `this` is NOT person here
  greet() { return `Hi ${this.name}`; },  // method shorthand: correct
};
console.log(person.greetArrow()); // "Hi undefined"
console.log(person.greet());      // "Hi Asha"
```

2. **Constructors** — arrows cannot be used with `new`:

```js
const Foo = () => {};
// new Foo(); // TypeError: Foo is not a constructor
```

3. **Functions that need `arguments`** — arrows don't have it (use rest parameters, section 7).

4. **DOM event handlers where you need `this` to be the element** — use a regular function, or use `event.currentTarget` instead.

### 3.5 Functions are values

Because functions are values, you can store them, pass them, and return them. This is the foundation for callbacks (next section) and closures (section 9).

```js
const operations = {
  add: (a, b) => a + b,
  sub: (a, b) => a - b,
};
console.log(operations["add"](2, 3)); // 5

function applyTwice(fn, value) {
  return fn(fn(value));
}
console.log(applyTwice((n) => n * 3, 2)); // 18
```

### 3.6 Default parameters

```js
const greet = (name = "stranger", punctuation = "!") => `Hello, ${name}${punctuation}`;
console.log(greet());              // "Hello, stranger!"
console.log(greet("Deepak"));      // "Hello, Deepak!"
console.log(greet(undefined, "?")); // "Hello, stranger?"  — undefined triggers default
console.log(greet(null));          // "Hello, null!"      — null does NOT
```

---

## 4. Callbacks

### 4.1 What a callback is

A **callback** is a function you hand to another function, to be called later. That is the whole idea. It exists because JavaScript is single-threaded: instead of *waiting*, you say "when X is done, run this."

Two flavours:

- **Synchronous callbacks** — run immediately during the call (`map`, `filter`, `forEach`, `sort`).
- **Asynchronous callbacks** — run later, after something finishes (`setTimeout`, events, network, file I/O).

### 4.2 Synchronous callbacks

```js
const nums = [1, 2, 3, 4];
nums.forEach((n, i) => console.log(i, n));

// Writing your own higher-order function
function repeat(times, action) {
  for (let i = 0; i < times; i++) {
    action(i);
  }
}
repeat(3, (i) => console.log("Run", i)); // Run 0, Run 1, Run 2
```

### 4.3 Asynchronous callbacks and the event loop

```js
console.log("1: start");

setTimeout(() => {
  console.log("3: timeout callback");
}, 0);

console.log("2: end");
// Output order: 1, 2, 3
```

Even with a delay of `0`, the callback runs **after** the current code finishes. The browser (or Node) puts it in a queue; the **event loop** picks it up once the call stack is empty. This is why you can never write:

```js
let data;
setTimeout(() => { data = "loaded"; }, 100);
console.log(data); // undefined — the callback hasn't run yet
```

### 4.4 Error-first callbacks (the Node convention)

```js
function readConfig(callback) {
  setTimeout(() => {
    const ok = Math.random() > 0.5;
    if (!ok) return callback(new Error("Failed to read"), null);
    callback(null, { theme: "dark" });
  }, 10);
}

readConfig((err, config) => {
  if (err) {
    console.log("Error:", err.message);
    return;
  }
  console.log("Config:", config);
});
```

Convention: the **first** argument is the error (or `null`), the **second** is the result.

### 4.5 Callback hell

Chaining dependent async steps nests deeper and deeper:

```js
function step(label, cb) {
  setTimeout(() => {
    console.log(label);
    cb();
  }, 10);
}

step("fetch user", () => {
  step("fetch orders", () => {
    step("fetch items", () => {
      step("render", () => {
        console.log("done");
      });
    });
  });
});
```

Problems: hard to read, hard to handle errors in one place, hard to run things in parallel. **Promises** and **async/await** (covered when you work with the Fetch API) were created to fix exactly this. For now, understand callbacks as the underlying mechanism.

### 4.6 Common callback pitfalls

**Losing `this`:**

```js
class Logger {
  prefix = "[log]";
  print(msg) { console.log(this?.prefix, msg); }
}
const logger = new Logger();
setTimeout(logger.print, 0, "oops");                 // [undefined?] — `this` lost
setTimeout((m) => logger.print(m), 0, "ok");         // [log] ok
setTimeout(logger.print.bind(logger), 0, "also ok"); // [log] also ok
```

**Calling the callback twice, or never:** document and guarantee that a callback is invoked exactly once.

**Passing the wrong thing:**

```js
console.log(["1", "2", "3"].map(parseInt));  // [1, NaN, NaN] — map passes (value, index, array)
console.log(["1", "2", "3"].map((s) => parseInt(s, 10))); // [1, 2, 3]
```

`parseInt` takes a second argument (the radix), and `map` supplies the index as that argument. Wrapping in an arrow function makes the arguments explicit.

---

## 5. Objects

### 5.1 Creating objects

An object is a collection of **key–value pairs** (properties).

```js
const user = {
  name: "Deepak",
  age: 24,
  skills: ["python", "fastapi"],
  address: { city: "Hyderabad", country: "India" },
  greet() {                        // method shorthand
    return `Hi, I'm ${this.name}`;
  },
};

console.log(user.name);            // dot notation
console.log(user["age"]);          // bracket notation
console.log(user.address.city);    // nested
console.log(user.greet());         // "Hi, I'm Deepak"
```

Use brackets when the key is dynamic or not a valid identifier:

```js
const key = "name";
console.log(user[key]);            // "Deepak"

const headers = { "Content-Type": "application/json" };
console.log(headers["Content-Type"]);
```

### 5.2 Property shorthand and computed keys

```js
const name = "Asha";
const age = 30;
const person = { name, age };       // same as { name: name, age: age }
console.log(person);

const field = "email";
const form = { [field]: "a@b.com", [`${field}Verified`]: false };
console.log(form); // { email: 'a@b.com', emailVerified: false }
```

### 5.3 Adding, changing, deleting, and checking

```js
const car = { brand: "Tata" };
car.model = "Nexon";               // add
car.brand = "Mahindra";            // update
delete car.model;                  // remove
console.log(car);                  // { brand: 'Mahindra' }

console.log("brand" in car);                    // true
console.log(Object.hasOwn(car, "brand"));       // true (modern replacement for hasOwnProperty)
console.log(car.color);                         // undefined
```

### 5.4 Looping over objects

```js
const scores = { math: 90, physics: 85, chem: 78 };

console.log(Object.keys(scores));     // ['math', 'physics', 'chem']
console.log(Object.values(scores));   // [90, 85, 78]
console.log(Object.entries(scores));  // [['math', 90], ['physics', 85], ['chem', 78]]

for (const [subject, mark] of Object.entries(scores)) {
  console.log(`${subject}: ${mark}`);
}

// Build an object from entries
const doubled = Object.fromEntries(
  Object.entries(scores).map(([k, v]) => [k, v * 2])
);
console.log(doubled); // { math: 180, physics: 170, chem: 156 }
```

Avoid `for...in` for plain data: it also walks inherited properties.

### 5.5 References vs copies

Objects are **reference types**. Assigning one to another variable copies the *reference*, not the object.

```js
const a = { n: 1 };
const b = a;       // same object
b.n = 99;
console.log(a.n);  // 99

console.log({} === {});  // false — two different objects
console.log(a === b);    // true  — same reference
```

Shallow vs deep copy:

```js
const original = { name: "x", tags: ["a", "b"], meta: { v: 1 } };

const shallow = { ...original };          // top level copied only
shallow.tags.push("c");
console.log(original.tags);               // ['a', 'b', 'c']  — shared!

const deep = structuredClone(original);   // true deep copy
deep.tags.push("d");
console.log(original.tags);               // still ['a', 'b', 'c']
```

`structuredClone` handles nested objects, arrays, dates, maps and sets, but not functions. `JSON.parse(JSON.stringify(x))` is a cruder alternative that drops functions, `undefined`, and dates-as-dates.

### 5.6 `this` inside methods

```js
const wallet = {
  balance: 100,
  spend(amount) {
    this.balance -= amount;
    return this.balance;
  },
};
console.log(wallet.spend(30)); // 70

const spendFn = wallet.spend;
// spendFn(10); // TypeError in strict mode / NaN — `this` is no longer wallet
```

`this` depends on the call site: `wallet.spend()` → `this` is `wallet`; a detached `spendFn()` → not.

### 5.7 Getters, setters, and classes (brief)

```js
const temp = {
  _c: 25,
  get fahrenheit() { return this._c * 9 / 5 + 32; },
  set fahrenheit(f) { this._c = (f - 32) * 5 / 9; },
};
console.log(temp.fahrenheit); // 77
temp.fahrenheit = 212;
console.log(temp._c);         // 100
```

Classes are syntax over the same prototype system:

```js
class Account {
  #balance = 0;                    // private field
  constructor(owner) { this.owner = owner; }
  deposit(n) { this.#balance += n; return this; }
  get balance() { return this.#balance; }
}
const acc = new Account("Deepak").deposit(50).deposit(25);
console.log(acc.balance); // 75
// console.log(acc.#balance); // SyntaxError — truly private
```

### 5.8 Merging objects

```js
const defaults = { theme: "light", lang: "en", debug: false };
const overrides = { theme: "dark", debug: true };
const settings = { ...defaults, ...overrides };   // later wins
console.log(settings); // { theme: 'dark', lang: 'en', debug: true }
```

### 5.9 JSON

```js
const payload = { id: 1, tags: ["a", "b"], active: true };
const text = JSON.stringify(payload);
console.log(text);                 // '{"id":1,"tags":["a","b"],"active":true}'
console.log(JSON.parse(text));     // back to an object
console.log(JSON.stringify(payload, null, 2)); // pretty-printed
```

JSON is a *text format*; it supports strings, numbers, booleans, null, arrays, and objects, but not functions, `undefined`, or `Date` objects (dates become strings).

---

## 6. Destructuring

Destructuring **unpacks** values from arrays or properties from objects into variables, in one statement. It exists because "pull out three fields" otherwise takes three lines of repetitive code.

### 6.1 Object destructuring

```js
const user = { id: 7, name: "Deepak", city: "Hyderabad" };

// Without destructuring
const id1 = user.id;
const name1 = user.name;

// With destructuring
const { id, name } = user;
console.log(id, name); // 7 'Deepak'
```

Rename, default, and combine:

```js
const { name: userName, role = "viewer" } = user;
console.log(userName); // 'Deepak'
console.log(role);     // 'viewer' — default used because user.role is undefined
```

Nested:

```js
const order = { id: 1, customer: { name: "Asha", address: { city: "Pune" } } };
const { customer: { name: customerName, address: { city } } } = order;
console.log(customerName, city); // Asha Pune
```

Careful: a nested pattern like this does **not** create a `customer` variable. It only creates the leaf variables.

### 6.2 Array destructuring

Position matters, not names.

```js
const rgb = [255, 128, 0];
const [r, g, b] = rgb;
console.log(r, g, b); // 255 128 0

const [first, , third] = ["a", "b", "c"];  // skip with an empty slot
console.log(first, third); // a c

const [head, ...tail] = [1, 2, 3, 4];
console.log(head, tail);   // 1 [2, 3, 4]

const [x = 10, y = 20] = [5];
console.log(x, y);         // 5 20
```

Swapping variables without a temp:

```js
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q); // 2 1
```

### 6.3 In function parameters (very common)

```js
// Named "options" style — order doesn't matter, defaults are easy
function createUser({ name, age = 18, role = "user" } = {}) {
  return { name, age, role };
}
console.log(createUser({ name: "Asha" }));          // { name: 'Asha', age: 18, role: 'user' }
console.log(createUser({ name: "Raj", age: 40 }));
console.log(createUser());                          // survives no argument thanks to `= {}`

// Destructure in an arrow callback
const people = [{ name: "A", age: 20 }, { name: "B", age: 30 }];
console.log(people.map(({ name }) => name)); // ['A', 'B']
```

Returning multiple values (a pattern React's `useState` uses):

```js
function minMax(arr) {
  return [Math.min(...arr), Math.max(...arr)];
}
const [lo, hi] = minMax([4, 9, 1, 7]);
console.log(lo, hi); // 1 9
```

### 6.4 Destructuring with `Map` and `entries`

```js
const prices = { apple: 30, mango: 80 };
for (const [fruit, price] of Object.entries(prices)) {
  console.log(`${fruit} costs ${price}`);
}
```

### 6.5 Pitfalls

```js
// Destructuring null/undefined throws
// const { a } = null; // TypeError: Cannot destructure property 'a' of 'null'
const { a } = null ?? {};   // safe fallback
console.log(a);             // undefined

// Statement starting with { is parsed as a block — wrap in parentheses
let m, n;
({ m, n } = { m: 1, n: 2 });
console.log(m, n); // 1 2
```

---

## 7. Spread and rest

Same `...` symbol, opposite jobs, decided by position:

- **Rest** — *collects* multiple items into one array/object (appears on the **receiving** side: function params, destructuring patterns).
- **Spread** — *expands* an array/object into individual items (appears on the **sending** side: function calls, array/object literals).

### 7.1 Rest parameters

```js
function sum(...numbers) {          // numbers is a real array
  return numbers.reduce((a, b) => a + b, 0);
}
console.log(sum(1, 2, 3, 4)); // 10

function log(level, ...messages) {  // rest must be last
  console.log(`[${level}]`, messages.join(" "));
}
log("INFO", "server", "started"); // [INFO] server started
```

Rest replaces the old `arguments` object, which was array-like but not an array and doesn't exist in arrow functions.

### 7.2 Rest in destructuring

```js
const [first, ...others] = [10, 20, 30, 40];
console.log(first, others); // 10 [20, 30, 40]

const { id, ...rest } = { id: 1, name: "A", age: 5 };
console.log(id, rest);      // 1 { name: 'A', age: 5 }
```

Handy for **omitting** a property immutably:

```js
const { password, ...safeUser } = { name: "A", password: "secret", role: "admin" };
console.log(safeUser); // { name: 'A', role: 'admin' }
```

### 7.3 Spread with arrays

```js
const a = [1, 2];
const b = [3, 4];

console.log([...a, ...b]);        // [1, 2, 3, 4]
console.log([0, ...a, 99, ...b]); // [0, 1, 2, 99, 3, 4]

const copy = [...a];              // shallow copy
copy.push(100);
console.log(a);                   // [1, 2] — unchanged

console.log(Math.max(...[3, 9, 2])); // 9 — spread into function arguments
console.log([..."hey"]);             // ['h', 'e', 'y'] — works on any iterable
console.log([...new Set([1, 1, 2, 3, 3])]); // [1, 2, 3] — dedupe
```

### 7.4 Spread with objects

```js
const base = { a: 1, b: 2 };
const extended = { ...base, c: 3 };
console.log(extended); // { a: 1, b: 2, c: 3 }

// Order matters: later keys overwrite earlier ones
console.log({ ...base, b: 99 }); // { a: 1, b: 99 }
console.log({ b: 99, ...base }); // { b: 2, a: 1 } — base overwrote b
```

### 7.5 Immutable updates — the core use case

Instead of mutating, build a *new* value. This is how state is updated in React, so get comfortable now.

```js
const state = {
  user: { name: "Asha", age: 30 },
  todos: [{ id: 1, done: false }, { id: 2, done: false }],
};

// Update a nested field
const next1 = { ...state, user: { ...state.user, age: 31 } };

// Add to array
const next2 = { ...state, todos: [...state.todos, { id: 3, done: false }] };

// Update one item in an array
const next3 = {
  ...state,
  todos: state.todos.map((t) => (t.id === 2 ? { ...t, done: true } : t)),
};

// Remove from array
const next4 = { ...state, todos: state.todos.filter((t) => t.id !== 1) };

console.log(state.user.age);       // 30 — original untouched
console.log(next1.user.age);       // 31
console.log(next3.todos);
```

Spread is a **shallow** copy, which is why you spread each level you change.

---

## 8. Array methods

Arrays are the data structure you'll use most. Learn to think in terms of **transformations** rather than manual loops.

### 8.1 Categories at a glance

| Category | Methods |
|---|---|
| Mutate the original | `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`, `fill` |
| Return a new array | `map`, `filter`, `slice`, `concat`, `flat`, `flatMap`, `toSorted`, `toReversed`, `toSpliced`, `with` |
| Return a single value | `reduce`, `find`, `findLast`, `findIndex`, `findLastIndex`, `at`, `indexOf`, `includes` |
| Return a boolean | `some`, `every`, `includes`, `Array.isArray` |
| Just iterate | `forEach` |

**Why it matters:** mutating methods surprise other code holding the same array. Prefer the non-mutating ones in app/state code. `toSorted`, `toReversed`, `toSpliced`, and `with` (all ES2023) are the non-mutating twins of `sort`, `reverse`, `splice`, and index assignment.

We'll use this data throughout:

```js
const products = [
  { id: 1, name: "Keyboard", price: 1500, category: "tech",  stock: 10 },
  { id: 2, name: "Mouse",    price: 700,  category: "tech",  stock: 0  },
  { id: 3, name: "Notebook", price: 120,  category: "paper", stock: 50 },
  { id: 4, name: "Pen",      price: 20,   category: "paper", stock: 200 },
  { id: 5, name: "Monitor",  price: 9000, category: "tech",  stock: 3  },
];
```

### 8.2 `forEach` — do something for each item

```js
products.forEach((p) => console.log(p.name));
```

It returns `undefined`, can't be `break`-ed, and doesn't wait for async callbacks. Use it only for side effects. For transformations use `map`.

### 8.3 `map` — transform every item (1 in → 1 out)

```js
const names = products.map((p) => p.name);
console.log(names); // ['Keyboard', 'Mouse', 'Notebook', 'Pen', 'Monitor']

const withTax = products.map((p) => ({ ...p, priceWithTax: p.price * 1.18 }));
console.log(withTax[0].priceWithTax); // 1770

// Callback receives (item, index, array)
console.log(["a", "b"].map((x, i) => `${i}:${x}`)); // ['0:a', '1:b']
```

Rule: the output array always has the **same length** as the input.

### 8.4 `filter` — keep only matching items

```js
const inStock = products.filter((p) => p.stock > 0);
console.log(inStock.map((p) => p.name)); // ['Keyboard', 'Notebook', 'Pen', 'Monitor']

const cheapTech = products.filter((p) => p.category === "tech" && p.price < 2000);
console.log(cheapTech.map((p) => p.name)); // ['Keyboard', 'Mouse']
```

Remove falsy values:

```js
console.log([0, 1, "", "a", null, undefined, 2].filter(Boolean)); // [1, 'a', 2]
```

### 8.5 `reduce` — fold everything into one value

```js
const totalValue = products.reduce((acc, p) => acc + p.price * p.stock, 0);
console.log(totalValue); // 15000 + 0 + 6000 + 4000 + 27000 = 52000
```

How it runs: `acc` starts at the initial value (`0`); for each item, the callback returns the **new** `acc`.

Group by (a very common use):

```js
const byCategory = products.reduce((groups, p) => {
  (groups[p.category] ??= []).push(p.name);
  return groups;
}, {});
console.log(byCategory); // { tech: ['Keyboard', 'Mouse', 'Monitor'], paper: ['Notebook', 'Pen'] }
```

Count occurrences:

```js
const votes = ["a", "b", "a", "c", "a", "b"];
const tally = votes.reduce((t, v) => ({ ...t, [v]: (t[v] ?? 0) + 1 }), {});
console.log(tally); // { a: 3, b: 2, c: 1 }
```

**Always pass an initial value.** Without it, an empty array throws a TypeError, and the first element is silently used as the accumulator.

> Modern note: `Object.groupBy(array, fn)` (ES2024) does grouping in one call where supported. Check your runtime before relying on it; `reduce` works everywhere.

### 8.6 `find`, `findIndex`, `findLast`, `findLastIndex`

```js
console.log(products.find((p) => p.price > 1000));        // Keyboard object (first match)
console.log(products.find((p) => p.price > 99999));       // undefined
console.log(products.findIndex((p) => p.name === "Pen")); // 3
console.log(products.findLast((p) => p.category === "tech").name); // 'Monitor'
console.log(products.findLastIndex((p) => p.category === "tech")); // 4
```

`filter(...)[0]` also works but scans the whole array; `find` stops at the first hit.

### 8.7 `some`, `every`, `includes`

```js
console.log(products.some((p) => p.stock === 0));    // true — at least one
console.log(products.every((p) => p.price > 0));     // true — all
console.log([1, 2, 3].includes(2));                  // true
console.log([NaN].includes(NaN));                    // true  (indexOf would give -1)
```

### 8.8 `sort` and `toSorted`

Default `sort` compares as **strings**:

```js
console.log([10, 9, 1, 100].sort());              // [1, 10, 100, 9]  <- wrong for numbers
console.log([10, 9, 1, 100].sort((a, b) => a - b)); // [1, 9, 10, 100]
console.log([10, 9, 1, 100].sort((a, b) => b - a)); // [100, 10, 9, 1]
```

Comparator rule: return negative → `a` first, positive → `b` first, zero → keep order.

```js
const byPriceAsc = products.toSorted((a, b) => a.price - b.price);
console.log(byPriceAsc.map((p) => p.name)); // ['Pen', 'Notebook', 'Mouse', 'Keyboard', 'Monitor']
console.log(products[0].name);              // 'Keyboard' — original untouched

// Strings: use localeCompare
console.log(["éclair", "apple", "Zebra"].toSorted((a, b) => a.localeCompare(b)));
```

`sort()` mutates the original; `toSorted()` returns a copy. Same pairing for `reverse`/`toReversed`.

### 8.9 `slice`, `splice`, `toSpliced`, `with`

```js
const letters = ["a", "b", "c", "d", "e"];

console.log(letters.slice(1, 3));   // ['b', 'c'] — end index excluded, no mutation
console.log(letters.slice(-2));     // ['d', 'e']

const copy = [...letters];
const removed = copy.splice(1, 2);  // MUTATES: remove 2 items starting at index 1
console.log(removed, copy);         // ['b', 'c'] ['a', 'd', 'e']

console.log(letters.toSpliced(1, 2, "X")); // ['a', 'X', 'd', 'e'] — non-mutating
console.log(letters.with(0, "Z"));         // ['Z', 'b', 'c', 'd', 'e']
console.log(letters.at(-1));               // 'e'
```

### 8.10 `flat`, `flatMap`

```js
console.log([1, [2, [3, [4]]]].flat());          // [1, 2, [3, [4]]]
console.log([1, [2, [3, [4]]]].flat(Infinity));  // [1, 2, 3, 4]

const sentences = ["hello world", "foo bar"];
console.log(sentences.flatMap((s) => s.split(" "))); // ['hello', 'world', 'foo', 'bar']
```

### 8.11 `Array.from`, `Array.of`, and friends

```js
console.log(Array.from("abc"));                       // ['a', 'b', 'c']
console.log(Array.from({ length: 5 }, (_, i) => i * 2)); // [0, 2, 4, 6, 8]
console.log(Array.from(new Set([1, 1, 2])));          // [1, 2]
console.log(Array(3).fill(0));                        // [0, 0, 0]
console.log(Array.of(7));                             // [7]
```

`Array.from` is also how you turn a DOM `NodeList` into a real array so `map` and `filter` work.

### 8.12 Chaining — where it clicks

```js
const report = products
  .filter((p) => p.stock > 0)                 // only available
  .map((p) => ({ name: p.name, value: p.price * p.stock }))
  .toSorted((a, b) => b.value - a.value)      // biggest first
  .slice(0, 3);                               // top 3

console.log(report);
// [{ name: 'Monitor', value: 27000 }, { name: 'Keyboard', value: 15000 }, { name: 'Notebook', value: 6000 }]
```

Each step takes an array and returns an array, so steps compose like a pipeline. Read it top to bottom as a sentence: "available products → shape them → sort → take three."

### 8.13 Choosing the right method

| You want to… | Use |
|---|---|
| Change each element | `map` |
| Keep some elements | `filter` |
| Compute one value from many | `reduce` |
| Get the first match | `find` |
| Check if any/all match | `some` / `every` |
| Do a side effect only | `forEach` or a `for...of` loop |
| Stop early with `break` | `for...of` (callbacks can't `break`) |

### 8.14 Common mistakes

```js
// 1. Forgetting to return in a braced arrow
console.log([1, 2].map((x) => { x * 2; }));   // [undefined, undefined]
console.log([1, 2].map((x) => { return x * 2; })); // [2, 4]

// 2. Using map just for side effects (wasted array) — use forEach / for...of

// 3. Mutating inside map
const bad = products.map((p) => { p.price = 0; return p; }); // mutates originals!

// 4. sort() on numbers without a comparator
```

---

## 9. Closures

### 9.1 Scope first

JavaScript uses **lexical (static) scope**: which variables a function can see is decided by **where the function is written**, not where it is called. Scopes nest, and a name lookup walks outward: local → enclosing → … → global. This chain is the **scope chain**.

```js
const globalVar = "global";

function outer() {
  const outerVar = "outer";
  function inner() {
    const innerVar = "inner";
    console.log(innerVar, outerVar, globalVar); // all visible
  }
  inner();
  // console.log(innerVar); // ReferenceError — inner scope is not visible outward
}
outer();
```

### 9.2 What a closure is

A **closure** is a function bundled together with references to the variables of the scope where it was created. In other words, a function that *remembers* its surroundings even after the outer function has finished.

```js
function makeGreeter(greeting) {
  return function (name) {          // inner function closes over `greeting`
    return `${greeting}, ${name}!`;
  };
}

const sayHello = makeGreeter("Hello");
const sayNamaste = makeGreeter("Namaste");

console.log(sayHello("Deepak"));    // Hello, Deepak!
console.log(sayNamaste("Deepak"));  // Namaste, Deepak!
```

`makeGreeter` has returned, so you'd expect `greeting` to be gone. It isn't: each returned function carries its own `greeting`. Closures are created every time a function is created.

### 9.3 Private state

JavaScript has long lacked private variables on plain functions; closures provide them.

```js
function createCounter() {
  let count = 0;                    // not accessible from outside

  return {
    increment() { count++; return count; },
    decrement() { count--; return count; },
    value() { return count; },
  };
}

const c1 = createCounter();
const c2 = createCounter();
c1.increment();
c1.increment();
c2.increment();
console.log(c1.value(), c2.value()); // 2 1  — independent state
console.log(c1.count);               // undefined — truly hidden
```

This is the **module pattern**: data lives in the closure; only the returned methods can touch it.

### 9.4 Function factories

```js
const multiplier = (factor) => (n) => n * factor;
const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5), triple(5)); // 10 15

// Configurable validators
const minLength = (min) => (str) => str.length >= min;
const isValidPassword = minLength(8);
console.log(isValidPassword("short"));        // false
console.log(isValidPassword("longenough123")); // true
```

### 9.5 Closures and callbacks

```js
function setupButton(label) {
  let clicks = 0;
  return function onClick() {       // would be passed to addEventListener
    clicks++;
    console.log(`${label} clicked ${clicks} time(s)`);
  };
}
const handler = setupButton("Save");
handler(); // Save clicked 1 time(s)
handler(); // Save clicked 2 time(s)
```

Event handlers, timers, and promise callbacks all work through closures: they remember the variables around them.

### 9.6 Memoization — caching with a closure

```js
function memoize(fn) {
  const cache = new Map();
  return (arg) => {
    if (cache.has(arg)) {
      console.log("cache hit");
      return cache.get(arg);
    }
    const result = fn(arg);
    cache.set(arg, result);
    return result;
  };
}

const slowSquare = (n) => n * n;
const fastSquare = memoize(slowSquare);
console.log(fastSquare(9)); // 81
console.log(fastSquare(9)); // cache hit, 81
```

### 9.7 Debounce — a real-world closure

```js
function debounce(fn, delay) {
  let timeoutId;                    // remembered between calls
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

const search = debounce((q) => console.log("searching:", q), 200);
search("j");
search("ja");
search("jav");   // only this one fires, 200ms after the last call
```

### 9.8 The loop pitfall, explained

```js
// var — one `i` shared by all closures
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log("var:", i), 0);   // 3, 3, 3
}

// let — each iteration has its own `j`
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log("let:", j), 0);   // 0, 1, 2
}
```

Closures capture **variables, not values**. With `var` there is one variable that ends at `3`. With `let` there are three separate variables.

Same effect: closure sees the *current* value of a variable at call time, not the value at creation time:

```js
let message = "first";
const show = () => console.log(message);
message = "second";
show(); // "second"
```

### 9.9 Memory considerations

A closure keeps its captured variables alive as long as the function itself is reachable. That's usually fine, but large objects captured by long-lived callbacks (listeners you never remove, intervals you never clear) can leak memory.

```js
const id = setInterval(() => console.log("tick"), 1000);
// later:
clearInterval(id);   // release the callback and everything it captured
```

---

## 10. ES modules

### 10.1 Why modules

Before modules, scripts shared one global scope, so names collided and load order was fragile. **ES modules (ESM)** give every file its own scope. Only what you `export` is visible to others, and other files pull it in explicitly with `import`. Properties of ES modules worth knowing:

- Each file is its own scope; top-level variables are **not** global.
- Modules always run in **strict mode**.
- A module is evaluated **once** and cached; every importer shares the same instance (a singleton).
- `import`/`export` declarations are resolved **before** code runs (static structure), which lets tools do tree-shaking and static analysis.

### 10.2 Named exports

```js
// math.js
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export const multiply = (a, b) => a * b;

// Or export at the bottom
function subtract(a, b) { return a - b; }
export { subtract };
```

```js
// app.js
import { add, multiply, PI } from "./math.js";
console.log(add(2, 3), multiply(2, 3), PI);
```

Names in the braces must match the exported names. Rename on either side with `as`:

```js
// math.js
export { subtract as minus };

// app.js
import { add as plus, minus } from "./math.js";
```

### 10.3 Default exports

A module can have one default export, imported **without braces** under any name you like.

```js
// logger.js
export default function log(message) {
  console.log(`[LOG] ${message}`);
}
```

```js
// app.js
import log from "./logger.js";     // name is your choice
import myLogger from "./logger.js"; // same function
```

Mixed:

```js
// api.js
export default class ApiClient {}
export const BASE_URL = "https://api.example.com";

// app.js
import ApiClient, { BASE_URL } from "./api.js";
```

**Named vs default:** named exports are explicit and autocomplete-friendly, and renaming is deliberate. Default exports let importers pick any name, which can hurt consistency across a codebase. Many style guides prefer named exports for utilities and reserve default for "the one main thing" in a file (e.g. a component).

### 10.4 Namespace imports and re-exports

```js
// import everything as one object
import * as math from "./math.js";
console.log(math.add(1, 2), math.PI);

// utils/index.js — re-export to build a single entry point
export { add, multiply } from "./math.js";
export { default as log } from "./logger.js";
export * from "./strings.js";
```

```js
// elsewhere: one import line instead of many
import { add, log } from "./utils/index.js";
```

### 10.5 Live bindings

Imports are **read-only live views** of the exporting module's variables, not copies.

```js
// counter.js
export let count = 0;
export function increment() { count++; }

// app.js
import { count, increment } from "./counter.js";
console.log(count); // 0
increment();
console.log(count); // 1 — reflects the change
// count = 5;       // TypeError — importers cannot assign
```

### 10.6 Using modules in the browser

```html
<script type="module" src="./app.js"></script>
```

Things that change with `type="module"`:

- The script is **deferred** by default (runs after the HTML is parsed).
- Paths must be real URLs/relative paths **including the file extension**: `"./math.js"`, not `"./math"`.
- Modules are fetched over HTTP with CORS rules, so opening `index.html` via `file://` often fails. Use a dev server (`npx serve`, `python -m http.server`, or Vite).
- Top-level `await` works inside modules.

```js
// top-level await in a module
const res = await fetch("/data.json");
const data = await res.json();
```

### 10.7 Using modules in Node.js

Either name the file `.mjs`, or set `"type": "module"` in `package.json`:

```json
{
  "name": "my-app",
  "type": "module"
}
```

Then:

```js
import fs from "node:fs/promises";
import { add } from "./math.js";     // extension required in Node ESM too
```

Older Node code uses CommonJS (`require` / `module.exports`). You will meet both in the wild; new code should use ESM.

```js
// CommonJS (legacy)
const fs = require("fs");
module.exports = { add };
```

### 10.8 Dynamic `import()`

Static `import` must be at the top level and use a string literal. `import()` is an expression that returns a **Promise** of the module namespace object, so you can load code conditionally or on demand.

```js
// Load only when needed (lazy loading)
button.addEventListener("click", async () => {
  const { renderChart } = await import("./chart.js");
  renderChart();
});

// Conditional loading
const lang = "hi";
const messages = await import(`./i18n/${lang}.js`);

// Without await
import("./math.js").then((mod) => console.log(mod.add(1, 2)));
```

Why bother: bundlers split dynamically imported modules into separate files, so users download heavy code only when they use it. Use it when it actually helps, not everywhere.

### 10.9 Import maps and bare specifiers (brief)

A **bare specifier** is a name without a path, like `import confetti from "canvas-confetti"`. Node resolves these from `node_modules`. Browsers can't without help; either use a bundler/dev server such as **Vite**, or an import map:

```html
<script type="importmap">
  { "imports": { "canvas-confetti": "https://cdn.jsdelivr.net/npm/canvas-confetti@1/+esm" } }
</script>
<script type="module">
  import confetti from "canvas-confetti";
  confetti();
</script>
```

### 10.10 Circular imports and side effects

If `a.js` imports `b.js` and `b.js` imports `a.js`, you can hit the TDZ for not-yet-initialised exports. Avoid cycles by moving shared code into a third module.

`import "./setup.js";` (no bindings) runs the module only for its side effects.

### 10.11 A small module layout

```
src/
  main.js              ← entry point
  api/
    users.js           ← export async function fetchUsers()
  utils/
    format.js          ← export const formatPrice = ...
    index.js           ← re-exports
  components/
    userList.js
```

```js
// utils/format.js
export const formatPrice = (n) =>
  new Intl.NumberFormat("en-IN", { style: "currency", currency: "INR" }).format(n);

// main.js
import { formatPrice } from "./utils/format.js";
console.log(formatPrice(1500)); // ₹1,500.00
```

---

## 11. Putting it all together

A small, self-contained program using every concept above: `const`/`let`, template strings, arrows, callbacks, objects, destructuring, spread/rest, array methods, closures, and module-style organisation.

```js
// ---- data (would normally come from a module or an API) ----
const orders = [
  { id: 1, customer: "Asha", items: [{ sku: "KB", qty: 1, price: 1500 }, { sku: "MS", qty: 2, price: 700 }], status: "paid" },
  { id: 2, customer: "Ravi", items: [{ sku: "PN", qty: 10, price: 20 }], status: "pending" },
  { id: 3, customer: "Asha", items: [{ sku: "MN", qty: 1, price: 9000 }], status: "paid" },
];

// ---- pure helpers (arrow functions, array methods) ----
const orderTotal = ({ items }) =>
  items.reduce((sum, { qty, price }) => sum + qty * price, 0);

const withTotal = (order) => ({ ...order, total: orderTotal(order) });

const groupBy = (arr, keyFn) =>
  arr.reduce((acc, item) => {
    const key = keyFn(item);
    (acc[key] ??= []).push(item);
    return acc;
  }, {});

// ---- closure: a configurable filter factory ----
const hasStatus = (status) => (order) => order.status === status;

// ---- pipeline ----
const paidSummary = orders
  .filter(hasStatus("paid"))
  .map(withTotal)
  .toSorted((a, b) => b.total - a.total)
  .map(({ id, customer, total }) => `#${id} ${customer}: ₹${total}`);

console.log(paidSummary);
// [ '#3 Asha: ₹9000', '#1 Asha: ₹2900' ]

const totalsByCustomer = Object.entries(groupBy(orders.map(withTotal), (o) => o.customer))
  .map(([customer, list]) => [customer, list.reduce((s, o) => s + o.total, 0)]);

console.log(Object.fromEntries(totalsByCustomer));
// { Asha: 11900, Ravi: 200 }

// ---- callback + closure: a tiny event emitter ----
function createEmitter() {
  const handlers = new Map();
  return {
    on(event, cb) {
      handlers.set(event, [...(handlers.get(event) ?? []), cb]);
    },
    emit(event, ...args) {
      (handlers.get(event) ?? []).forEach((cb) => cb(...args));
    },
  };
}

const bus = createEmitter();
bus.on("order:paid", ({ id, customer }) => console.log(`Send receipt for #${id} to ${customer}`));
orders.filter(hasStatus("paid")).forEach((o) => bus.emit("order:paid", o));
```

### Practice ideas

1. Rewrite the pipeline above so `orderTotal` applies a 10% discount to any order over ₹5000, without mutating the data.
2. Write `pipe(...fns)` that composes functions left-to-right using rest parameters and `reduce`.
3. Build `once(fn)` using a closure so `fn` runs at most one time.
4. Split section 11 into three modules (`data.js`, `helpers.js`, `main.js`) and use named exports.
5. Implement `groupBy` and `countBy` using `reduce`, then compare with `Object.groupBy` if your runtime supports it.
6. Given `const users = [{ name, age, address: { city } }]`, produce an array of `"name — city"` strings using destructuring in the `map` parameter list.

---

## Quick reference

**Declarations:** `const` by default → `let` when reassigning → no `var`.
**Equality:** `===` always; `??` for defaults; `?.` to safely reach into data.
**Functions:** arrows for callbacks; method shorthand for object methods; default and rest parameters instead of `arguments`.
**Objects/arrays:** destructure to read, spread to copy/merge, never mutate shared state when you can build a new value.
**Array pipeline:** `filter → map → reduce`; prefer `toSorted`/`toReversed`/`toSpliced`/`with` over their mutating counterparts.
**Closures:** functions remember the variables from where they were created; they power private state, factories, memoization, debounce, and every event handler.
**Modules:** one file = one scope; named exports by default; `import()` for lazy loading; `type="module"` in the browser, `"type": "module"` in Node.

## References

- MDN — `let`, `const`, temporal dead zone: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let
- MDN — Closures: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures
- MDN — Dynamic `import()`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import
- MDN — JavaScript modules guide: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
- MDN — Array reference: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
- javascript.info — The Modern JavaScript Tutorial: https://javascript.info
