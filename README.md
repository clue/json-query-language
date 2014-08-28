# JSON Query Language

A simple query language that can be expressed as simple JSON strings.
*Experimental*.

* Simple to write and produce
* Simple to consume, parse and evaluate
* Lightweight and open

## Examples

Match users with the given properties:

```json
{ "age": 20, "registered": true }
```

Match users with the "country" property set to any in the list:

```json
{ "country": ["BE", "NL"] }
``` 

Match users with IDs below 100:

```json
{ "id": { "$lt": 100 }}
```

## Syntax definition

See [syntax definition](SYNTAX.md).

## License

MIT, see LICENSE file.

Acknowledgements, heavily inspired by:

* [NeDB](https://github.com/louischatriot/nedb#finding-documents)
* [TaffyDB](http://www.taffydb.com/writingqueries)
