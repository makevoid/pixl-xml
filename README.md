# Overview

This module provides a lightweight, fast, easy-to-use XML parser which generates a simplified object / array tree.  This can be very useful for parsing XML configuration files and the like.  It is 100% pure JavaScript and has no dependencies.

* Pure JavaScript, no dependencies
* Very fast parser (About 5X faster than [xml2js](https://www.npmjs.com/package/xml2js))
* Low memory usage (About 60% less than [xml2js](https://www.npmjs.com/package/xml2js))
* Fully synchronous operation, no callbacks
* Can parse XML strings or load from files
* Can preserve or flatten attributes
* Can convert all keys to lower-case
* Can serialize objects back to pretty-printed XML

# Usage

Use [npm](https://www.npmjs.com/) to install the module:

```
	npm install pixl-xml
```

Then use `require()` to load it in your code:

```javascript
	var XML = require('pixl-xml');
```

Parse some XML by passing a string to `XML.parse()`:

```javascript
	var xml_string = '<?xml version="1.0"?><Document>' + 
		'<Simple>Hello</Simple>' + 
		'<Node Key="Value">Complex</Node>' + 
		'</Document>';
	
	var doc = XML.parse( xml_string );
	console.log( doc );
```

That would produce an object like the following:

```javascript
	{
	  "Simple": "Hello",
	  "Node": {
	    "Key": "Value",
	    "_Data": "Complex"
	  }
	}
```

Note that the outermost element is omitted from the object (`<Document>` in this case).  Also, for complex elements that have both attributes (or sub-elements) *and* plain data, the plain data goes into a `_Data` property.  Multiple elements with the same name at the same level are converted to an array.

You can also pass in a path to an XML file on disk, and it'll be loaded, then parsed:

```javascript
	var config = XML.parse( 'conf/config.xml' );
	console.log( config );
```

Parsing errors will be thrown as exceptions, so you'd better wrap `parse()` calls in a try/catch for safety:

```javascript
	var doc = null;
	try {
		doc = XML.parse( 'my_xml_file.xml' );
	}
	catch (err) {
		console.log("XML Parser Error: " + err);
	}
	console.log( doc );
```

## Options

You can pass an optional 2nd argument to `parse()`, which can be an object containing any of the following properties:

### preserveAttributes

This optional property, when set to `true`, will cause all XML attributes to be kept separate in their own sub-object called `_Attribs` for each element.  For example, consider this snippet:

```javascript
	var xml_string = '<?xml version="1.0"?><Document>' + 
		'<Simple>Hello</Simple>' + 
		'<Node Key="Value">Complex</Node>' + 
		'</Document>';
	
	var doc = XML.parse( xml_string, { preserveAttributes: true } );
	console.log( doc );
```

With the `preserveAttributes` flag set to true, this would produce the following object:

```javascript
	{
	  "Simple": "Hello",
	  "Node": {
	    "_Attribs": {
	      "Key": "Value"
	    },
	    "_Data": "Content"
	  }
	}
```

Notice the `Key` attribute of the `<Node>` element is now kept in an `_Attribs` sub-object.  The only real purpose for this is if you expect to write the XML back out again (see [Composing XML](#composing-xml) below).

### lowerCase

This optional property, when set to `true`, will cause all keys to be lower-cased as the XML is parsed.  This affects both elements and attributes.  Example:

```javascript
	var xml_string = '<?xml version="1.0"?><Document>' + 
		'<Simple>Hello</Simple>' + 
		'<Node Key="Value">Complex</Node>' + 
		'</Document>';
	
	var doc = XML.parse( xml_string, { lowerCase: true } );
	console.log( doc );
```

With the `lowerCase` flag set to true, this would produce the following object:

```javascript
	{
	  "simple": "Hello",
	  "node": {
	    "key": "Value",
	    "_data": "Content"
	  }
	}
```

Note that the values themselves are not touched -- only the keys are lower-cased.

## Composing XML

To compose XML back to a string, call `XML.stringify()` and pass in your pre-parsed XML object, and an outer wrapper element name.  It helps to parse using the [preserveAttributes](#preserveattributes) option for this, as it will honor the `_Attribs` sub-objects and convert them back into real XML attributes.  Example:

```javascript
	var xml_string = XML.stringify( doc, 'Document' );
	console.log( xml_string );
```

This would produce something like:

```xml
	<?xml version="1.0"?>
	<Document>
		<Node Key="Value">Content</Node>
		<Simple>Hello</Simple>
	</Document>
```

Note that elements and attributes may lose their original ordering, as hashes have an undefined key order.  However, to keep things consistent, they are both alphabetically sorted when serialized.

## Utility Functions

Here are a few utility functions you can use:

### encodeEntities

```
	STRING encodeEntities( STRING )
```

This function will take a string, and encode the three standard XML entities, ampersand (`&`), left-angle-bracket (`<`) and right-angle-bracket (`>`), into their XML-safe counterparts.  It returns the result.  Example:

```javascript
	var text = '<Hello>&<There>';
	console.log( XML.encodeEntities(text) );
	// Would output: &lt;Hello&gt;&amp;&lt;There&gt;
```

### encodeAttribEntities

```
	STRING encodeAttribEntities( STRING )
```

This function does basically the same thing as [encodeEntities](#encodeentities), but it also includes encoding for single-quotes (`'`) and double-quotes (`"`).  It is used for encoding an XML string for composing into an attribute value.  It returns the result.  Example:

```javascript
	var text = '<Hello>"&"<There>';
	console.log( XML.encodeAttribEntities(text) );
	// Would output: &lt;Hello&gt;&quot;&amp;&quot;&lt;There&gt;
```

### decodeEntities

```
	STRING decodeEntities( STRING )
```

This function decodes all the standard XML entities back into their original characters.  This includes ampersand (`&`), left-angle-bracket (`<`), right-angle-bracket (`>`), single-quote (`'`) and double-quote (`"`).  It is used when parsing XML element and attribute values.  Example:

```javascript
	var text = '&lt;Hello&gt;&quot;&amp;&quot;&lt;There&gt;';
	console.log( XML.decodeEntities(text) );
	// Would output: <Hello>"&"<There>
```

### alwaysArray

```
	ARRAY alwaysArray( MIXED )
```

This function will wrap anything passed to it into an array and return the array, unless the item passed is already an array, in which case it is simply returned verbatim.

```javascript
	var arr = XML.alwaysArray( maybe_array );
```

### hashKeysToArray

```
	ARRAY hashKeysToArray( OBJECT )
```

This function returns all the hash keys as an array.  Useful for sorting and then iterating over the sorted list.

```javascript
	var my_hash = { foo: "bar", baz: 12345 };
	var keys = XML.hashKeysToArray( my_hash ).sort();
	
	for (var idx = 0, len = keys.length; idx < len; idx++) {
		var key = keys[idx];
		// do something with key and my_hash[key]
	}
```

### isaHash

```
	BOOLEAN isaHash( MIXED )
```

This function returns `true` if the provided argument is a hash (object), `false` otherwise.

```javascript
	var my_hash = { foo: "bar", baz: 12345 };
	var is_hash = XML.isaHash( my_hash );
```

### isaArray

```
	BOOLEAN isaArray( MIXED )
```

This function returns `true` if the provided argument is an array (or is array-like), `false` otherwise.

```javascript
	var my_arr = [ "foo", "bar", 12345 ];
	var is_arr = XML.isaArray( my_arr );
```

### numKeys

```
	INTEGER numKeys( OBJECT )
```

This function returns the number of keys in the specified hash.

```javascript
	var my_hash = { foo: "bar", baz: 12345 };
	var num = XML.numKeys( my_hash ); // 2
```

### firstKey

```
	STRING firstKey( OBJECT )
```

This function returns the first key of the hash when iterating over it.  Note that hash keys are stored in an undefined order.

```javascript
	var my_hash = { foo: "bar", baz: 12345 };
	var key = XML.firstKey( my_hash ); // foo or baz
```

# License

Copyright (c) 2004 - 2015 Joseph Huckaby

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.