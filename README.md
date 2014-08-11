# JSON Query Language

A simple query language that can be expressed as simple JSON strings.

* Simple to write and produce
* Simple to consome, parse and evaluate
* Lightweight and open

## Examples

Match users with the given properties:

```json
{ "age": 20, "registerd": true }
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

## History

Heavily inspired by:

* NeDB: https://github.com/louischatriot/nedb#finding-documents
* TaffyDB: http://www.taffydb.com/writingqueries

## License

MIT, see LICENSE file.

Acknowledgements, heavily inspired by NeDB and TaffyDB.
