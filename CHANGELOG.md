# Changelog

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

