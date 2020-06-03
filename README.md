# Deep Path Properties in Record Literals

ECMAScript proposal for deep paths properties for [Record literals](https://github.com/tc39/proposal-record-tuple).

**Author:**

- Rick Button (Bloomberg)

**Champions:**

- Rick Button (Bloomberg)
- Robin Ricard (Bloomberg)

**Advisors:**

- Dan Ehrenberg (Igalia)

**Stage:** 1, Reached at June 2020 TC39

# Overview

[Record literals](https://github.com/tc39/proposal-record-tuple) sometimes include deeply nested structures, but the syntax for describing them (either as a fresh value, or based on a previous value via spread syntax) can be cumbersome and/or verbose. Deep path properties for `Record` literals provides a solution to this problem, by introducing a new syntax for describing deeply nested structures in a more succinct and readable way.

## Examples

These examples demonstrate a possible syntax for deep path properties for `Record` literals.

```js
const state1 = #{
    counters: #[
        #{ name: "Counter 1", value: 1 },
        #{ name: "Counter 2", value: 0 },
        #{ name: "Counter 3", value: 123 },
    ],
    metadata: #{
        lastUpdate: 1584382969000,
    },
};

const state2 = #{
    ...state1,
    counters[0].value: 2,
    counters[1].value: 1,
    metadata.lastUpdate: 1584383011300,
};

assert(state2.counters[0].value === 2);
assert(state2.counters[1].value === 1);
assert(state2.metadata.lastUpdate === 1584383011300);

// As expected, the unmodified values from "spreading" state1 remain in state2.
assert(state2.counters[2].value === 123);
```

In the previous example, two counters are incremented, and the "lastUpdate" time is updated in the new record `state2`.

Without deep path properties, `state2` can be created in a few different ways:

```js
// With records/tuples and recursive usage of spread syntax
const state2 = #{
    ...state1,
    counters: #[
        #{
            ...state1.counters[0],
            value: 2,
        },
        #{
            ...state1.counters[1],
            value: 1,
        },
        ...state1.counters,
    ],
    metadata: #{
        ...state1.metadata,
        lastUpdate: 1584383011300,
    },
}

// With Immer (and regular objects)
const state2 = Immer.produce(state1, draft => {
    draft.counters[0].value = 2;
    draft.counters[1].value = 1;
    draft.metadata.lastUpdate = 1584383011300;
});

// With Immutable.js (and regular objects)
const immutableState = Immutable.fromJS(state1);
const state2 = immutableState
    .setIn(["counters", 0, "value"], 2)
    .setIn(["counters", 1, "value"], 1)
    .setIn(["metadata", "lastUpdate"], 1584383011300);
```

### A Simple Example

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

No, the semantics for deep path properties for objects are much less clear than they are for `Record`, where (because `Records` are immutable) the semantics are much simpler.


#### What happens if the deep path property does not exist in the value that is spread?

A `TypeError` is thrown. For example:

```js
const one = #{ a: #{} };

#{ ...one, a.b.c: "foo" }; // throws TypeError

#{ ...one, a.b[0]: "foo" }; // also throws TypeError
```

One might expect that a record or tuple somehow "materializes" in these cases, to fill in the path. However, when deep path property syntax is used with spread syntax, there can be ambiguities in what kind of value to "materialize" if the value doesn't already exist. In the latter example, both of the following expansions seem like valid answers:

```js
#{ a: #{ b: #[123] } }

#{ a: #{ b: #{ 0: 123 } } }
```

To keep things simple and minimal, attempting to use a deep path property where the path doesn't already exist in the spread value (whether there is this kind of ambiguity or not) throws a `TypeError`.

#### What happens if a deep path property attempts to set a non-number-like key on a Tuple

A `TypeError` is thrown. For example:

```js
const one = #{ a: #[1,2,3] };

#{ ...one, a.foo: 4 }; // throws TypeError
```

`Tuples` cannot have non-number-like keys, as they are immutable ordered lists of values and do not have the concept of a "property" (just like a `number` doesn't have properties). If you attempt to create a `Record` literal with deep path property syntax that would create a `Tuple` with a non-number-like key, a `TypeError` is thrown.

See [issue #4](https://github.com/rickbutton/proposal-record-deep-spread/issues/4) for more discussion.
