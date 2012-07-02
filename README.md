# SerializeJs

SerializeJs allows to convert a generic JavaScript object graph into a more
primitive linear representation that can be stringified (as JSON). The
stringified representation can be deserialized and the object graph restored.

## SerializeJs !== JSON

The approach of SerializeJs is notebly different to `JSON.stringify(object)`
and `JSON.parse(stringifiedObject)`. With JSON only primitive data types and
simple object trees can be serialized. Consider this simple example:

```js
var obj1 = {}, obj2 = {};
obj1.ref = obj2;
obj2.ref = obj1;
JSON.stringify(obj1); // will raise error
```

The stringify call will raise an error since circular object structures are
not supported. Even if the application data structure itself requires no
circularity, simply referencing an object outside of the application data will
lead to problems:

```js
var obj = {doc: document};
JSON.serialize(obj); // error
```

## What is SerializeJs?

1. SerializeJs provides an algorithm for linearizing any JavaScript object
graph and restoring it.

2. SerializeJs provides (de)serialization hooks to give fine-granular control
over the (de)serialization process

### Usage for serialization

Using SerializeJs without any additions:

```js
var obj1 = {}, obj2 = {};
obj1.ref = obj2;
obj2.ref = obj1;
var serialized = SerializeJs.serialize(obj1);
```

The value of `serialized` then is:

```JSON
{
  "0": {
    "ref": {
      "__isSmartRef__": true,
      "id": 1
    }
  },
  "1": {
    "ref": {
      "__isSmartRef__": true,
      "id": 0
    }
  }
}
```

The circular object structure was converted into a list, the references
between `obj1` and `obj2` are made explicitly. This list structure can then be
converted into normal JSON. JSON is the default output format, however, any
other data representation such as XML would also be possible. Currently,
support only for JSON is implemented.

### Usage for deserialization

To deserialize the stringified representation of `obj1`:
```js
var obj1_deserialized = SerializeJs.deserialize(serialized);
obj1_deserialized.ref.ref === obj1_deserialized; // true
```

Note that the deserialized version is a deep copy of `obj`:

```js
obj1_deserialized === obj1; // false
obj1_deserialized.ref === obj1.ref; // false
```

If required, you can change that behavior to allow shallow copies or a mix of
deep and shallow copies.

### Customizing (de)serialization

When (de)serializing, plugin objects can be passed to the serializer that are
used during the store or restore process to change how objects are represented
or how objects created their from stringified representation.

The serialization interface:

```js
serializeObj: function(original) {},
additionallySerialize: function(original, persistentCopy) {},
deserializeObj: function(persistentCopy) {},
ignoreProp: function(obj, propName, value) {},
```

The deserialization interface:
```js
ignorePropDeserialization: function(obj, propName, value) {},
afterDeserializeObj: function(obj) {},
deserializationDone: function() {},
serializationDone: function(registry) {},
```
