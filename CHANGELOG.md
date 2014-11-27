# Changelog

## 0.4.0 (2014-11-27)

* Feature: Support [L2] matching multiple operators (#3)

* Feature: New `$contains` comparator (#9)

* Feature: Support root matching (#13)

* BC break: Empty `$or` lists and [L2] objects do *never* match (#10)

* Move double negation to L2 (#11)

* Move matching literal dots to L2 (#15)

* Improve formatting and links

## 0.3.0 (2014-11-04)

* Explicitly state that missing keys are to be handled as `NULL` values

* Move simple matching (scalar and list) to L2

* Remove boolean filters `true` and `false` (#5)

* Negation prefix `!` is now the preferred negation method
  * Move double-negation to L1
  * Move negation operator and combinator `$not` to L2 and explicitly describe its unfolding rules.

* Improve wording, avoid ambiguities

## 0.2.0 (2014-08-13)

* Feature / BC break: Empty `$and` and `$or` lists are now allowed and both *always* match.

* Feature: Supporting nested keys adds a requirement for literal dots
  
  ```json
{
    "dotted\\.key": "value"
}
```

* All custom extension points have been moved to dedicated L3

## 0.1.0 (2014-08-11)

First tagged release

