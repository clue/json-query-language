# Syntax

The following is a specification of the supported syntax.

## About

### Layers

The specifiation is split into several layers.

* Layer 1 (L1) is the default and defines the base syntax that must be supported.
* Layer 2 (L2) is a folded syntax that is more convenient to produce when matching multiple keys (in particular when the filters are not machine-produced).
  This syntax can be transformed into the more explicit base syntax (unfolding).
  The folded syntax is often assumed to be easier to produce and consume. However, it often turns out to be limited and/or ambiguous when it comes to more complex nested filters.
  An implementation of this specification is expected to implement this syntax (though not strictly demanded).
* Layer 3 (L3) defines custom extension points.
  An implementation may chose to extend the mandated base syntax by providing custom extensions in these situations.

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

### Nested keys

Nested keys are supported using dot notation like this:

```json
{
    "name.first": "Peter"
}
```

This filter matches every object that has a "name" sub-object that has first=Peter.

#### Literal dot

In the case that your object keys actually include a dot in the name,
you can use the `\` prefix to escape the dot.
Because the backslash has to be escaped by another backslash, the resulting filter looks like this:

```json
{
    "dotted\\.key": "value"
}
```

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

An empty list will never match.

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

#### L3 Common comparators

An implementation may chose to define some of the common operators as a fallback:

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

The `$not` comparator is a shorthand for negated matching.

##### L2 Not scalar

The `$not` comparator can be used for scalar values like this:

```json
{ "id": { "$not": 100 } }
```

This filter matches every object that does NOT have id=100.

The above example can also be written explicitly like this:

```json
{ "id": { "!$is": 100 } }
```

##### L2 Not list

The `$not` comparator can be used for lists like this:

```json
{ "id": { "$not": [ 100, 200 ] } }
```

This filter matches every object that does NOT have (id=100 OR id=200).

The above example can also be written explicitly like this:

```json
{ "id": { "!$in": [ 100, 200 ] } }
```

#### L3 Double negation

Prefixing every comparator with a `!` results in a negated comparator. Because
of this, it's legal to double-negate comparators like this:

```json
{ "id": { "!!!$is": 100 } }
```

Double-negation is effectively a NO-OP. Because of this, the above example is equivalent to this:

```
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

Because of these unfolding rules, an empty object will always match.

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

An empty `$AND` list will always match.
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

Because of these unfolding rules, an empty object will always match.

### OR

Expects a list of filters like this:

```json
{ "$or": [ filter, … ] }
```

Matches it one of the given filters in the list matches.
Does not match if none of the given filters in the list match.

An empty `$OR` list will always match.
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

Because of these unfolding rules, an empty object will always match.

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

Because of these unfolding rules, an empty list will never match.

#### L2 NOT prefix

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

#### L2 NOT multiple keys

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

Because of these unfolding rules, an empty object will never match.

### L3 Additional combinators

An implementation may chose to implement additional combinators like `$xor`, `$xand` and others.

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
