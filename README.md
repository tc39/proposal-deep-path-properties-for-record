# Deep Path Properties in Record Literals

ECMAScript proposal for deep paths properties for [Record literals](https://github.com/tc39/proposal-record-tuple).

**Author:**

- Richard Button (Bloomberg)

**Champions:**

- Richard Button (Bloomberg)
- Robin Ricard (Bloomberg)

**Advisors:**

- Dan Ehrenberg (Igalia)

**Stage:** 0

# Overview

[Record literals](https://github.com/tc39/proposal-record-tuple) sometimes include deeply nested structures, but the syntax for describing them (either as a fresh value, or based on a previous value via spread syntax) can be cumbersome and/or verbose. Deep path properties for `Record` literals provides a solution to this problem, by introducing a new syntax for describing deeply nested structures in a more succient and readable way.

## Examples

These examples demonstrate a possible syntax for deep path properties for `Record` literals.

```js
const rec = #{ a.b.c: 123 };
assert(rec === #{ a: #{ b: #{ c: 123 }}});
```

### Computed Deep Path Property Keys

```js
const rec = #{ ["a"]["b"]["c"]: 123 }
assert(rec === #{ a: #{ b: #{ c: 123 }}});
```

It is possible to mix dot syntax with computed keys.

```js
const b = "b";
const rec = #{ ["a"][b].c: 123 }
assert(rec === #{ a: #{ b: #{ c: 123 }}});
```

### Combining Deep Path Properties with Spread

```js
const one = #{
    a: 1,
    b: #{
        c: #{
            d: 2,
            e: 3,
        }
    }
};
const two = #{
    b.c.d: 4,
    ...one,
};

assert(one.b.c.d === 2);
assert(two.b.c.d === 4);
```

It is possible to traverse through `Tuples`.

```js
const one = #{
    a: 1,
    b: #{
        c: #[2, 3, 4, #[5, 6]]
    },
}
const two = #{
    b.c[3][1]: 7,
    ...one,
};

assert(two.b.c === #[2, 3, 4, #[5, 7]]);
```

# FAQ

#### Does deep path properties syntax support objects?

No, the semantics for deep path propertiess for objects are much more unclear than they are for `Record`, where (because `Records` are immutable) the semantics are much simpler.


#### What happens if the deep path property does not exist in the value that is spread?

A TypeError is thrown. For example:

```js
const one = #{ a: #{} };

#{ a.b.c: "foo", ...one }; // throws TypeError
```

When deep path property syntax is used with spread syntax, there can be ambiguities in what kind of value to "materialize" if the value doesn't already exist. In the previous example, both of the following expansions seem like valid answers:

```js
#{ a: #{ b: #[123] } }

#{ a: #{ b: #{ 0: 123 } } }
```

Because of this ambiguity, attempting to use a deep path property where the path doesn't already exist in the spread value, or where there an ambiguity in what to create (like the number-like key example above), a `TypeError` should be thrown.

#### What happens if a deep path property attempts to set a non-number-like key on a Tuple

A TypeError is thrown. For example:

```js
const one = #{ a: #[1,2,3] };

#{ ...one, a.foo: 4 }; // throws TypeError
```

`Tuples` cannot have non-number-like keys, as they are immutable ordered lists of values and do not have the concept of a "property" (just like a `number` doesn't have properties). If you attempt to create a `Record` literal with deep path property syntax that would create a `Tuple` with a non-number-like key, a `TypeError` is thrown.

See [issue #4](https://github.com/rickbutton/proposal-record-deep-spread/issues/4) for more discussion.
