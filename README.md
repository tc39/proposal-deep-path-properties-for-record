# Deep Spread for Records

ECMAScript proposal for deep spread for [Records](https://github.com/tc39/proposal-record-tuple).

**Author:**

- Richard Button (Bloomberg)

**Champions:**

- Richard Button (Bloomberg)
- Robin Ricard (Bloomberg)

**Advisors:**

- Dan Ehrenberg (Igalia)

**Stage:** 0

# Overview

Because [`Records`](https://github.com/tc39/proposal-record-tuple) are deeply immutable data structures, the only way to "manipulate" them is to create a new `Record` with the same properties as the old record with spread syntax (or `Record.assign`).

If deep path updates are required (updating a value nested deeply inside a `Record`), then the syntax is
cumbersome, because a spread is needed at each level of nesting. Deep path spread syntax for `Record` is a possible solution to this problem, which allows the user to "copy" a `Record` and update deeply nested
properties within it without cumbersome syntax.

These examples demonstrate a possible syntax for deep spreads on `Records`.

```js
const one = #{
    a: 1,
    b: {
        c: {
            d: 2,
            e: 3,
        }
    }
};
const two = #{
    b.c.d: 4,
    ...one,
};

console.log(one.b.c.d); // 2
console.log(two.b.c.d); // 4
```

Traversal through tuples:

```js
const one = #{
    a: 1,
    b: #{
        c: #[2, 3, 4, [5, 6]]
    },
}
const two = #{
    b.c[3][1]: 7,
    ...one,
};

console.log(two.b.c); // #[2, 3, 4, [5, 7]]
```

Computed properties:

```js
const one = #{
    a: {
        b: {
            c: {
                foo: "bar",
            }
        }
    },
    d: {
        bill: "ted",
    },
};

const two = #{
    ["a"]["b"]["c"]["foo"]: "baz",
    ...one,
};

// can be mixed
const three = #{
    ["a"].b["c"].foo: "baz",
    ...one,
};
```

Question! What happens if the deep path does not exist in the value that is spread?

Example:

```js
const one = { a: 1 };

const two = { b.c: 2, ...one };

// Does this fail, or create a Record that looks like:
// #{ a: 1, b: { c: 2 } }
```
