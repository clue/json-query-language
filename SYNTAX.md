# Syntax

The following is a specification of the supported syntax.

## About

### Layers

The specifiation is split into several layers.

* Layer 1 (L1) is the default and defines the base syntax that must be supported.
* Layer 2 (L2) is convenience syntax that an implementation of this specification
  may chose to implement. This syntax can be transformed into the more explicit
  base syntax.

### Example data

This document pro

```json
[
    { "id": 100, "name": "Test", "age": 20 },
    { "id": 200, "name": "Peter", "age": 25 }
]
```

## Matching Values

### Matching equals

```json
{
    "id": 100
}
```

This filter matches every object that has id=100.

Nested keys. Use dot notation.

### Matching IN

```json
{
    "id": [
        100,
        200,
        300
    ]
}
```

This filter matches every object that has any of id=100, id=200 OR id=300.

### Comparators

The default comparision checks for strict equal.
The above example can be expressed like this:

```json
{
    "id": {
        "$is": 100
    }
}
```

You can also use the following list of comparators:

| comparator | meaning |
|------------|---------|
| `$is`      | strictly equal (type and value) |
| `$in`      | in (either of) |
| `$lt`      | less than |
| `$lte`     | less than or equal |
| `$gt`      | greater than |
| `$gte`     | greater than or equal |

```json
{ "id": { "$gt": 200 }}
```

#### L2 Comparators

Some of the common operators are also defined as a fallback:

```json
{ "id": { ">=": 100 } }
{ "id": { "$gt": 100 } }
```

### Negation

Every comparator can be negated by prefixing it with `!` like this:

```json
{
    "id" {
        "!$is": 200
    }
}
```

This filter matches every object that does NOT have id=200.

#### L2 Not negation

`$not` is a fallback for `!$is` / `!$in`.

```json
{ "id": { "$not": 100 } }
{ "id": { "!$is": 100 } }

{ "id": { "$not": [ 100, 200 ] } }
{ "id": { "!$in": [ 100, 200 ] } }
```

#### L2 Double negation

Prefixing every comprator with a `!` results in a negated comparator. Because
of this, it's legal to double-negate negated comparators like this:

```json
{ "id": { "!!!$is": 100 } }
{ "id": { "!$is": 100 } }
```

### L2 Matching multiple keys

```json
{
    "id": 100,
    "name": "Test"
}
```

Matches every object that has *both* id=100 *AND* name=Test.

The same matching and comparator rules as above apply.

See also following chapter about *Combining*.
The above example is a shorthand syntax for the following:

```json
{
    "$and": [
        {
            "id": 100
        },
        {
            "name": "Test"
        }
    ]
}
```

## Combining

### AND

Expects a list of filters like this:

```
{ "$and": [ filter, … ] }
```

Only matches if each and every of the given filters in the list do match.
Does not match if any of the given filters does not match.

```json
{
    "$and": [
        {
            "id": 100
        }
        {
            "name": "Test"
        }
    ]
}
```

An emtpy `$AND` statement is not allowed.
Providing only a single filter expression is supported.

#### L2 AND object

Also accepts an object like this:

```json
{
    "$and": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that has *both* id=100 *AND* name=Test.

The above example can also be written explicitly like this:

```json
{
    "$and": [
        {
            "id": 100
        },
        {
            "name": "Test"
        }
    ]
}
```

### OR

Expects a list of filters like this:

```json
{ "$or": [ filter, … ] }
```

Matches it one of the given filters in the list matches.
Does not match if none of the given filters in the list match.

An empty `$OR` statement is not allowed.
Prividing only a single filter expression is supported.

#### L2 OR object

Also accepts an object like this:

```json
{
    "$or": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that has *either* id=100 *OR* name=Test.

The above example can also be written explicitly like this:

```json
{
    "$or": [
        {
            "id": 100
        },
        {
            "name": "Test"
        }
    ]
}
```

### NOT

Expects a filter like this:

```json
{ "$not": filter }
```

Only matches if the filter does not match.
Does not match if the filter matches.

Accepts an object like this:

```json
{
    "$not": {
        "id": 100
    }
}
```

Matches every object that does NOT have id=100.

This is regularly equivalent to negated matching like this:

```json
{
    "id" : { "!$is": 100 }
}
```

Also, negating other combinators by prefixing with a "!" is supported:

```json
{
    "!$and": [ filter, filter ]
}
```

The above example can also be written explicitly like this:
```json
{
    "$not": {
        "$and": [ filter, filter ]
    }
}
```

#### L2 NOT list

Can also be used with a list of filters like this:

```json
{ "$not": [ filter, … ] }
```

Only matches if each and every of the given filters in the list do match.
Does not match if any of the given filters does not match.

```json
{
    "$not": [ filter, filter ]
}
```

The above example can also be written explicitly like this:
```json
{
    "$not": {
        "$and": [ filter, filter ]
    }
}
```

An empty `$NOT` statement list is not allowed.

##### L2 NOT multiple keys

Also accepts an object with multiple keys like this:

```json
{
    "$not": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that does NOT have (id=100 *AND* name=Test).

The above example can also be written explicitly like this:

```json
{
    "$not": {
        "$and": [
            {
                "id": 100
            },
            {
                "name": "Test"
            }
        ]
    }
}
```

An empty `$NOT` statement is not allowed.

## Additional

### Booleans

A filter that always matches:

```json
[ true ]
```

A filter that never matches:

```json
[ false ]
```

## Nesting

Nesting filters is supported.
