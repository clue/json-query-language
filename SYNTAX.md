# Syntax

The following is a specification of the supported syntax.

**Table of contents:**

* [About](#about)
  * [Layers](#layers)
    * [L1] [Base syntax](#l1-base-syntax)
    * [L2] [Folded syntax](#l2-folded-syntax)
    * [L3] [Custom extension](#l3-custom-extension)
  * [Example data](#example-data)
* [Matching Values](#matching-values)
  * [Basic matching](#basic-matching)
  * [Nested keys](#nested-keys)
    * [Literal dot](#literal-dot)
  * [Missing keys](#missing-keys)
  * [Negation](#negation)
  * [L2] [Matching scalar](#l2-matching-scalar)
  * [L2] [Matching list](#l2-matching-list)
  * [L2] [Matching multiple keys](#l2-matching-multiple-keys)
* [Operators](#operators)
  * [Comparators](#comparators)
    * [$is comparator](#is-comparator)
    * [$in comparator](#in-comparator)
    * [$lt comparator](#lt-comparator)
    * [$lte comparator](#lte-comparator)
    * [$gt comparator](#gt-comparator)
    * [$gte comparator](#gte-comparator)
    * [L2] [$not comparator](#l2-not-comparator)
      * [L2] [Not scalar](#l2-not-scalar)
      * [L2] [Not list](#l2-not-list)
    * [L3] [Additional comparators](#l3-additional-comparators)
    * [L3] [Common comparators](#l3-common-comparators)
  * [Combinators](#combinators)
    * [$and combinator](#and-combinator)
      * [$and list](#and-list)
      * [L2] [$and object](#l2-and-object)
    * [$or combinator](#or-combinator)
      * [$or list](#or-list)
      * [L2] [$or object](#l2-or-object)
    * [L2] [$not combinator](#l2-not-combinator)
      * [L2] [$not list](#l2-not-list-1)
      * [L2] [$not object](#l2-not-object)
    * [L3] [Additional combinators](#l3-additional-combinators)
    * [L3] [Common combinators](#l3-common-combinators)
* [Nesting](#nesting)

## About

### Layers

The specifiation is split into several layers.

#### [L1] Base syntax

Layer 1 (L1) is the default and defines the base syntax that must be supported.

#### [L2] Folded syntax

Layer 2 (L2) is a folded syntax that is more convenient to produce when matching multiple keys (in particular when the filters are not machine-produced).

This syntax can be transformed into the more explicit base syntax (unfolding).
The folded syntax is often assumed to be easier to produce and consume. However, it often turns out to be limited and/or ambiguous when it comes to more complex nested filters.

An implementation of this specification is expected to implement this syntax (though not strictly demanded).

#### [L3] Custom extension

Layer 3 (L3) defines custom extension points.
An implementation may choose to extend the mandated base syntax by providing custom extensions in these situations.

### Example data

This document pro

```json
[
    { "id": 100, "name": "Test", "age": 20 },
    { "id": 200, "name": "Peter", "age": 25 }
]
```

## Matching Values

### Basic matching

The basic syntax for matching the value of a key to a given value always looks like this:

```json
{
    "key" : { "$comparator" : value }
}
```

The possible [comparators](#comparators) (among with examples) are defined in the [comparator section](#comparators).

### Nested keys

Nested keys are supported using dot notation like this:

```json
{
    "name.first": { "$comparator" : value }
}
```

This filter matches every object that has a "name" sub-object where the value of the key "first" matches the comparator.

#### Literal dot

In the case that your object keys actually include a dot in the name,
you can use the `\` prefix to escape the dot.
Because the backslash has to be escaped by another backslash, the resulting filter looks like this:

```json
{
    "dotted\\.key" : { "$comparator" : value }
}
```

### Missing keys

Accessing the value of a key that does not exist will always yield a `null` value.

```json
{
    "unknown" : { "$comparator" : value }
}
```

Will evaluate as if the key had a `null` value if the key "unknown" does not exist.
This will check if the value `null` matches against the value of the comparator.

There's currently no way to tell a non-existant key apart from a key that actually holds a null value. (see issue #1)

### Negation

Every comparator can be negated by prefixing it with `!` like this:

```json
{
    "key" : { "!$comparator": value }
}
```

This filter matches every object where the given comparator does NOT match the given value.

Prefixing every comparator with a `!` results in a negated comparator. Because
of this, it's legal to double-negate comparators like this:

```json
{
    "key": { "!!!$comparator": value }
}
```

Double-negation is effectively a NO-OP. Because of this, the above example is equivalent to this:

```json
{
    "key": { "!$comparator": value }
}
```

### [L2] Matching scalar

This convenient shortcut syntax allows one to leave out the [`$is` comparator](#is-comparator) for scalar values to compare against.

```json
{
    "id": 100
}
```

This is equivalent to the longer form

```json
{
    "id": {
        "$is" : 100
    }
}
```

This filter matches every object that has id=100.

### [L2] Matching list

This convenient shortcut syntax allows one to leave out the [`$in` comparator](#in-comparator) for a list of values to compare against.

```json
{
    "id": [
        100,
        200,
        300
    ]
}
```

This is equivalent to the longer form

```json
{
    "id": {
        "$in" : [
            100,
            200,
            300
        ]
    }
}
```

This filter matches every object that has either of id=100 OR id=200 OR id=300.

An empty list will never match.

### [L2] Matching multiple keys

```json
{
    "id": 100,
    "name": "Test"
}
```

Matches every object that has *both* id=100 *AND* name=Test.

The same matching and comparator rules as above apply.

See also following chapter about [combinators](#combinators).
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

## Operators

This query language supports two classes of operators further described below:

* [Comparators (comparison operators)](#comparators)
* [Combinators](#combinators)

### Comparators

You can use any of the following comparison operators (comparators)

#### $is comparator

The `$is` comparator checks if the value of the key is strictly equal (type and value) to the given value.

```json
{ "id": { "$is": 100 }}
```

This filter matches every object that has id=100.

Note that this comparators checks for strictly equal values and types.

```json
{ "id": { "$is": "100" }}
```

This filter matches every object that has an id attribute of type string and id=100.
In the above example, this does not match any object, because id is always of type number.

If you want to support multiple types, consider using the [`$in` comparator](#in-comparator) and explicitly list all possible types.

#### $in comparator

The `$in` comparator checks if the value of the key is "in" (either of) the given list of values.

```json
{ "id": { "$in": [100, 101, 102] }}
```

This filter matches every object that has either of id=100 OR id=101 OR id=102.

Note that this comparator checks for strictly equal values and types, just like the [`$is` comparator](#is-comparator).

```json
{ "id": { "$in": ["100", "101"] }}
```

This filter matches every object that has an id attribute of type string and a value of either id=100 or id=101.
In the above example, this does not match any object, because id is always of type number.

If you want to support multiple types, you can explicitly list all possible types like this:

```json
{ "registered": { "$in": [false, 0, null] }}
```

This matches every object that has either of registered=false OR registered=0 or registered=null.

An empty list will never match.

This comparator exclusively accepts an array of possible values, passing anything else (e.g. single scalar or object etc.) is a syntax error.

#### $lt comparator

The `$lt` comparator checks if the value of the key is "less than" the given value.

```json
{ "id": { "$lt": 100 }}
```

This filter matches every object that has an id of less than 100, i.e. it matches 99, but does not match 100.

#### $lte comparator

The `$lte` comparator checks if the value of the key is "less than or equal to" the given value.

```json
{ "id": { "$lte": 100 }}
```

This filter matches every object that has an id of less than or equal to 100, i.e. it matches 99, it matches 100 but does not match 101.

#### $gt comparator

The `$gt` comparator checks if the value of the key is "greater than" the given value.

```json
{ "id": { "$gt": 100 }}
```

This filter matches every object that has an id of greater than 100, i.e. it matches 101, but does not match 100.

#### $gte comparator

The `$gte` comparator checks if the value of the key is "greater than or equal to" the given value.

```json
{ "id": { "$gte": 100 }}
```

This filter matches every object that has an id of greater than or equal to 100, i.e. it matches 101, it matches 100, but does not match 99.

#### [L2] $not comparator

The `$not` comparator is a shorthand for negated matching.

It accepts either a scalar value or a list of values. Every other type (e.g. object etc.) is a syntax error.

##### [L2] Not scalar

The `$not` comparator can be used for scalar values like this:

```json
{ "id": { "$not": 100 } }
```

The above example can also be written explicitly like this:

```json
{ "id": { "!$is": 100 } }
```

This filter matches every object that does NOT have id=100.

##### [L2] Not list

The `$not` comparator can be used for lists like this:

```json
{ "id": { "$not": [ 100, 200 ] } }
```

The above example can also be written explicitly like this:

```json
{ "id": { "!$in": [ 100, 200 ] } }
```

This filter matches every object that does NOT have (id=100 OR id=200).

#### [L3] Additional comparators

An implementation may choose to define additional custom operators like `$contains`, `$regex`, `$starts`, `$ends` and others.

#### [L3] Common comparators

An implementation may choose to define some of the common operators as a fallback:

```json
{ "id": { ">=": 100 } }
{ "id": { "$gt": 100 } }
```

### Combinators

Combinators allow one to combine multiple filters to a bigger filter rule.

#### $and combinator

##### $and list

Expects a list of filters like this:

```json
{ "$and": [ filter, … ] }
```

Only matches if each and every of the given filters in the list do match.
Does not match if any of the given filters does not match.

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

An empty `$and` list will always match.
Providing only a single filter expression is supported.

##### [L2] $and object

Also accepts a folded L2 object like this:

```json
{
    "$and": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that has *both* id=100 *AND* name=Test.

The above example can be unfolded to the explicit L1 list form:

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

#### $or combinator

##### $or list

Expects a list of filters like this:

```json
{ "$or": [ filter, … ] }
```

Matches it one of the given filters in the list matches.
Does not match if none of the given filters in the list match.

An empty `$or` list will always match.
Prividing only a single filter expression is supported.

##### [L2] $or object

Also accepts a folded L2 object like this:

```json
{
    "$or": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that has *either* id=100 *OR* name=Test.

The above example can be unfolded to the explicit L1 list form:

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

#### [L2] $not combinator

Negating combinators is achieved by prefixing them with `!`.
The `$not` combinator is a convenience shorthand for `!$and`.

##### [L2] $not list

Expects a list of filter like this:

```json
{
    "$not": [ filter, filter ]
}
```

Only matches if one of the given filters in the list does NOT match.
Does not match if each and every of the given filters does match.

The above example can also be written like this:

```json
{
    "!$and": [ filter, filter ]
}
```

Because of these unfolding rules, an empty list will never match.

##### L2 $not object

Can also be used with a filter object like this:

```json
{
    "$not": filter
}
```

Only matches if the filter does not match.
Does not match if the filter matches.

Accepts an object like this:

```json
{
    "$not": {
        "id": {
            "$is" : 100
        }
    }
}
```

Matches every object that does NOT have id=100.

The above example can also be written like this:

```json
{
    "!$and" : {
        "id": {
            "$is": 100
        }
    }
}
```

This is equivalent to negated matching like this:

```json
{
    "id" : {
        "!$is": 100
    }
}
```

Also accepts a folded L2 object like this:

```json
{
    "$not": {
        "id": 100,
        "name": "Test"
    }
}
```

Matches every object that does NOT have (id=100 *AND* name=Test).

The above example can be unfolded to the explicit L1 basic matching form:

```json
{
    "$not": {
        "id": {
            "$is" : 100
        },
        "name": {
            "$is" : "Test"
        }
    }
}
```

Which can then be transformed to the explicit negation form:

```json
{
    "!$and": {
        "id": {
            "$is" : 100
        },
        "name": {
            "$is" : "Test"
        }
    }
}
```

Due to [De Morgan's laws](http://en.wikipedia.org/wiki/De_Morgan%27s_laws) this is equivalent to:

```json
{
    "$or": {
        "id": {
            "!$is" : 100
        },
        "name": {
            "!$is" : "Test"
        }
    }
}
```

Because of these unfolding rules, an empty object will never match.

#### [L3] Additional combinators

An implementation may choose to implement additional combinators like `$xor`, `$xnor` and others.

#### [L3] Common combinators

An implementation may choose to implement some common combinators.

A common alias of the L2 `$not` combinator:

```
{ "$nand" : filters }
{ "!$and" : filters }
```

Or the disjunctive equivalent of the L2 `$not` combinator:

```
{ "$nor" : filters }
{ "!$or" : filters }
```

Or common names for the negated possible L3 `$xor` combinator:

```
{ "$xnor" : filters }
{ "!$xor" : filters }
```

## Nesting

Nesting filters is supported.
