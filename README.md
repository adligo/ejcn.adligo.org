# Extensible JSON Classification Notation (EJCN)

### Abstract

EJCN is a simple, binary, hybrid [UTF-8 text](#utf8), context-free and content-sensitive grammar for encoding data. It is a data serialization format for files and messages similar to and inspired by [JSON](#json) [(lines)](#json-lines) and [XML](#xml). It is built around a simple header and body structure; however, it is recursive so that the body may have an EJCN structure as well.

### Structure Overview

EJCN is comprised of header and body sections. The header section is extremely formal; however, the body can be anything, including [JSON](https://www.json.org/), [XML](https://www.w3.org/XML/), pure binary, and even a child EJCN structure.

### Header Details

The header object instance is comprised of one or more lines, each line MUST contain valid [JSON](#json) similar to the [JSON lines](#json-lines) convention.  In addition, each line MUST be
terminated by a [UNIX Line Feed '\n'](#ascii), [ASCII](#ascii)/[UTF-8](#utf8) value of 10 (0x0A in [hexadecimal](#hexadecimal)). Each line MAY have an optimization integer or [Ten64](https://github.com/adligo/ten64.adligo.org) prefix for parsing, or MAY be nearly identical to the [JSON lines](#json-lines) model. However, this format has been optimized slightly for parsing when number prefixes exist. To clarify, the first character of each line in the header MUST be one of the following:

##### Header Line First Characters

* **0-9** [Modern Western Numerals](https://github.com/adligo/ten64.adligo.org#modern-western-numeral-system) are a number prefix which identify the number of bytes in this line, for single line entities, or multi-line comment.  The [Modern Western Numerals](https://github.com/adligo/ten64.adligo.org#modern-western-numeral-system) MUST represent an integer, and MUST be followed by either JSON or EJCN Comment start characters from this section.
<br/><br/>
* **#** The pound symbol identifies this number prefix as a [Ten64](#ten64) number, which identifies the number of bytes in this line for single line entities, or multi-line comment.  The [Ten64](https://github.com/adligo/ten64.adligo.org) number MUST represent an integer, and MUST be followed by either JSON or EJCN Comment start characters from this section.
<br/><br/>
* **{** The left curly brace identifies this line as a [JSON](#json) line.
<br/><br/>
* **[** The left square bracket identifies this line as a [JSON](#json) line.
<br/><br/>
* **//** The slash characters identifies this header line as a single line comment line.  This character was chosen because of the compatibility with the // code comment convention in many languages.
<br/><br/>
* **&lt;!--** The less than character identifies this header line as a multiple line comment line.  This indicates a HTML style comment.  Note the <b>'-->'</b> ending tag MUST be the last non whitespace text on the last line of the comment.

```
// one line comments go here
<!-- comments go here --> 
<!-- 
other comments go here
--> 
```

The purpose of including the number of bytes per line, either with [Ten64](https://github.com/adligo/ten64.adligo.org) or [Modern Western Numerals](https://github.com/adligo/ten64.adligo.org#modern-western-numeral-system), is simply an optimization for EJCN parsers. For example, the three in the following code identifies that the subsequent line with '3{}' only has 3 bytes;

In addition, the valid JSON MUST immediately follow any number prefix, [Modern Western Numerals](https://github.com/adligo/ten64.adligo.org#modern-western-numeral-system), or [Ten64](https://github.com/adligo/ten64.adligo.org). In other words, there must not be any white space between the prefix and the JSON. 

```
3{}
Plain Text Message

```

```
21{ "sender":"Bob" }
Plain Text Message

```

##### Use of ASCI-7/UTF-8

Although not a hard requirement, the use of only [ASCII-7](#ascii)/[UTF-8](#utf8) (NOT UTF-8 extended characters) is highly recommended in the header.  This simply maps bytes to characters and reduces processing time.

### Header Lines

A single line or list of lines is expected, however this can be changed with the [headers headerLines key](#header-lines).  Each header line MAY EITHER consist of valid JSON, or valid EJCN comments.  

### Header Keys

Although all header keys are optional, these are the main conventions.

##### bodyClass

This optional <b>'bodyClass'</b> key identifies a string which represents a [EJCN Schema fully qualified class name](https://github.com/adligo/ejcn_schemas.adligo.org#fully-qualified-class-loader-name-space-names).  This field identifies the how the bytes in the body MUST be interpreted.
The <b>'class'</b> key is recommended for data which MAY be of different classes, this is usually disk data.  However, some APIs may return or accept data from various classes, which would likely benefit from the <b>'bodyClass'</b> key.  Here the bodyClass is the class of the body, so only one class per header is allowed.

##### class

This optional <b>'class'</b> key identifies a string which represents a [EJCN Schema fully qualified class name](https://github.com/adligo/ejcn_schemas.adligo.org#fully-qualified-class-loader-name-space-names).  The <b>'class'</b> key is NOT recommended for API's which always deliver or accept the same class.  The <b>'class'</b> key is recommended for data which MAY be of different classes, this is usually disk data.  However, some APIs may return or accept data from various classes, which would likely benefit from the <b>'class'</b> key.  Here the class is the class of the header, so only one class per header is allowed.

##### headerLines

The optional <b>'headerLines'</b> key, a positive JSON integer identifies the integer number of lines with JSON that are part of the header.  Note, comment lines are considered part of the <b>'headerLines'</b> when they immediately follow the initial JSON line style header.  When <b>'headerLines'</b> is omitted, the default value is one.

The header may be comprise of multiple [JSON lines](#json-lines), each MAY have the optimization of a integer number prefix either using [Modern Western Numerals](https://github.com/adligo/ten64.adligo.org#modern-western-numeral-system) or [Ten64](https://github.com/adligo/ten64.adligo.org).

##### size

The optional <b>'size'</b> key, a positive integer identifies the number of bytes in the body. This can be particularly useful when transporting arbitrary bytes. In particular, for images and other binary data.  The <b>'size'</b> key is mutually exclusive with the keys <b>'size64','lines' and 'lines64'</b> keys, in other words: they MUST NOT be used at the same time.

##### size64

The optional <b>'size'</b> key identifies, a [Ten64 String](https://github.com/adligo/ten64.adligo.org) representing a positive integer, the number of bytes in the body. This can be particularly useful when transporting arbitrary bytes. In particular, for images and other binary data.  The <b>'size64'</b> key is mutually exclusive with the keys <b>'size','lines' and 'lines64'</b> keys, in other words: they MUST NOT be used at the same time.

##### lines

The optional <b>'lines'</b> key, a positive JSON integer, identifies the number of lines in a text formatted body.  The <b>'lines'</b> key is mutually exclusive with the keys <b>'size','size64' and 'lines64'</b> keys, in other words: they MUST NOT be used at the same time.

##### lines64

The optional <b>'lines64'</b> key, a [Ten64 String](https://github.com/adligo/ten64.adligo.org) representing a positive integer, identifies the number of lines in a text formatted body.  The <b>'lines64'</b> key is mutually exclusive with the keys <b>'size','size64' and 'lines'</b> keys, in other words: they MUST NOT be used at the same time.

### Extending the keys

It is generally recommended NOT to include some information about the data in the headers like "dataType":"JSON", but instead simply route your requests to parts of your application or services that know they are going to receive JSON, or a EJCN data tree.

# EJCN Sequences

EJCN bodies MAY be comprised of a list of EJCN segments.  However, when this is used all EJCN segments MUST ALWAYS include information about the length of the segment [i.e. [size](#size), [size64](#size64), [lines](#lines) or [lines64](#lines64)]!  Only the last EJCN segment in an EJCN sequence MAY ommit these fields.  Root EJCN segments MAY also omit these fields.

### Examples

The simplest EJCN data, with a empty header;

```
{}
Some plain text

```

Complex EJCN data, note how size is used with the email text command in order to split out binary that happens later in the nested EJCN body.

```
24{ "cmd":"sendEmail" }
37{ "part":"emailText", "size": 16 }
Some plain text
43{ "part":"emailAttachment", "size": 53 }
&@#LK!#J%LKHASR@#LKJ
-- not a real image #$%LK#J%^

```

A example with [Ten64](https://github.com/adligo/ten64.adligo.org), note the dollar sign $ after the pound symbol (#) in the first line turns into a 21. Also note, that all EJCM parsers are NOT likely to have support for [Ten64](https://github.com/adligo/ten64.adligo.org).

```
#${ "cmd":"sendData" }
{ "name": "George", 
  "age": 23,
  "height": 5.75
}

```

A example with an extended 3 line header.

```
{ "cmd":"sendData", "headers": 3 }
{ "dateSent": "2026-03-10" }
{ "sequenceNbr": 3 }
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>

```

A example with an optimized, extended 3 line header.

```
37{ "cmd":"sendData", "headers": 3 }
31{ "dateSent": "2026-03-10" }
23{ "sequenceNbr": 3 }
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
</note>

```

A example with an optimized, header with optimized comments.

```
23{ "cmd":"sendText" }
14//A comment
Hey you guys!
```

A example with an optimized, header optimized comments, note the J (45) from [Ten64](https://github.com/adligo/ten64.adligo.org) includes the return character bytes.

```
23{ "cmd":"sendText" }
#J<!--
a optimized multiple line comment 
-->
Hey you guys!
```

### Compatibility with other technologies.


EJCN is designed to be compatible with just about anything, including [REST](rest), [XML schemas](#xml), [FIX protocol encoding](#fix), [JSON schemas](#json-schemas), [JSON Lines](#json-lines), [CSV](#csv) files, [Google Protocol Buffers](#google-protocol-buffers), binary images, zip files that include some of the above in a simple format that is highly configurable and extensible.

### Transport Specific EJCN

[ESTN](https://github.com/adligo/estn.adligo.org) is the extension that is specific for the transport layer.

### EJCN Schemas

EJCN data can be more narrowly classified with schemas.  [EJCN Schemas](https://github.com/adligo/ejcn_schemas.adligo.org) greatly increased the clarity and ability to perform simple validations via [ECMA Script](#ecma-script) specific regular expressions.  [EJCN Schemas](https://github.com/adligo/ejcn_schemas.adligo.org), are encouraged to prevent injection and [OWASP DoS](#owasp-dos)/[DDoS](#owasp-dos) attacks vs services that accept or provide EJCN data.

### IANA Considerations

IANA number 1.3.6.1.4.1.33097.8.3, attempted to obtain file extension reservation <b>.ejcn</b>.

### Discussion Channel

 -[https://discord.gg/6BNgJwBMzd](https://discord.gg/6BNgJwBMzd)

### Commentary

EJCN started out as part of [ASBP (Asynchronous Services Bus Protocol)](https://datatracker.ietf.org/doc/draft-adligo-hybi-asbp/). Then for a time, I debated on whether it would use [classification markup notation CMN](https://github.com/adligo/cmn.adligo.org). 
Then for a few days it was synonymous with [ESTN](https://github.com/adligo/estn.adligo.org).
It was also, partially inspired by the [JSON lines project](https://jsonlines.org/).

On 2026-03-22 I was able to successfully submit a request to reserve the <b>.ejcn</b> file extension here, with the respective [values](iana_log/2026-03-22_ATTEMPT.md);

- [https://www.iana.org/form/media-types](https://www.iana.org/form/media-types)

# References 

###### Apache Kafka

- [Apache Kafka Official Website](https://kafka.apache.org/)
- [Wikipedia](https://en.wikipedia.org/wiki/Apache_Kafka)
- [Kafka on AWS](https://aws.amazon.com/what-is/apache-kafka/)
- [What is Kafka?](https://www.confluent.io/what-is-apache-kafka/)
- [Where is Kafka going?](https://www.youtube.com/watch?v=9CrlA0Wasvk)

###### ASCII

- [ASCII](https://www.ascii-code.com/)
- [Wikipedia](https://en.wikipedia.org/wiki/ASCII)
- [Table](https://www.asciitable.com/)

###### ECMA Script

- [ECMA International](https://ecma-international.org/publications-and-standards/standards/ecma-262/)
- [Wikipedia](https://en.wikipedia.org/wiki/ECMAScript)
- [Mozilla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/JavaScript_technologies_overview)


###### CSV

- [Wikipedia](https://en.wikipedia.org/wiki/Comma-separated_values)
- [What is a CSV File?](https://www.businessinsider.com/reference/what-is-csv-file)
- [Dr. Nesvit](https://www.youtube.com/watch?v=O0-T-LQgc_o)

###### FIX

FIX Trading Community, "What is FIX?", FIX Trading Community v.o.z., [Online]. Available: [https://www.fixtrading.org/what-is-fix/](https://www.fixtrading.org/what-is-fix/). [Accessed: 29-Mar-2026].

###### Google Protocol Buffers

- [Protobuf](https://protobuf.dev/)
- [Wikipedia](https://en.wikipedia.org/wiki/Protocol_Buffers)
- [Github](https://github.com/protocolbuffers/protobuf)
- [Medium](https://medium.com/javarevisited/what-are-protocol-buffers-and-why-they-are-widely-used-cbcb04d378b6)

###### Hexadecimal

- [Wikipedia](https://en.wikipedia.org/wiki/Hexadecimal)
- [Editor](https://cryptii.com/pipes/hex-decoder)
- [Math World](https://mathworld.wolfram.com/Hexadecimal.html)

###### HTTP

- [World Wide Web Consortium](https://www.w3.org/Protocols/)
- [Wikipedia](https://en.wikipedia.org/wiki/HTTP)
- [Cloudflare](https://www.cloudflare.com/learning/ddos/glossary/hypertext-transfer-protocol-http/)
- [Mozilla](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)
- [/1 rfc2616](https://datatracker.ietf.org/doc/html/rfc2616)
- [/2 rfc7540](https://httpwg.org/specs/rfc7540.html)

###### JSON

- [JSON.org](https://www.json.org/json-en.html)
- [Wikipedia](https://en.wikipedia.org/wiki/JSON)
- [W3schools](https://www.w3schools.com/whatis/whatis_json.asp)
- [Formatter](https://jsonformatter.org/)
- [Mozilla](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/JSON)

###### JSON Lines

- [Jsonlines.org/](https://jsonlines.org/)
- [Medium](https://medium.com/@sujathamudadla1213/difference-between-ordinary-json-and-json-lines-fc746f93d75e)
- [National Library of Medicine](https://www.ncbi.nlm.nih.gov/datasets/docs/v2/reference-docs/file-formats/metadata-files/why-jsonl/)
- [IBM](https://www.ibm.com/docs/en/taw/1.3.0?topic=analytics-json-versus-json-lines)

###### JSON Schemas

- [Json-schema.org](https://json-schema.org/)
- [Postman](https://blog.postman.com/what-is-json-schema/)
- [MongoDB](https://www.mongodb.com/resources/languages/json-schema-examples)
- [GitHub](https://cswr.github.io/JsonSchema/spec/definitions_references/)

###### Octal

###### OWasp DoS

- [OWasp DoS](https://owasp.org/www-community/attacks/Denial_of_Service)
- [NCSC](https://www.ncsc.gov.uk/collection/denial-service-dos-guidance-collection)
- [Cloudflare DDoS](https://www.cloudflare.com/learning/ddos/what-is-a-ddos-attack/)

###### Pub/Sub

- [GCP Docs Pub/Sub](https://docs.cloud.google.com/pubsub/docs/overview)
- [GCP Pub/Sub](https://cloud.google.com/pubsub)
- [Wikipedia](https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern)
- [AWS Pub/Sub](https://aws.amazon.com/what-is/pub-sub-messaging/)
- [Azure Pub/Sub](https://azure.microsoft.com/en-us/products/web-pubsub)
- [Microsoft](https://learn.microsoft.com/en-us/azure/architecture/patterns/publisher-subscriber)

###### REST

- [Original Paper](https://roy.gbiv.com/pubs/dissertation/top.htm)
- [Wikipedia](https://en.wikipedia.org/wiki/REST)
- [IBM](https://www.ibm.com/think/topics/rest-apis)
- [Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding)
- [Understanding the REST Style](https://www.youtube.com/watch?v=w5j2KwzzB-0)

###### Sockets

- [Wikipedia](https://en.wikipedia.org/wiki/Network_socket)
- [IBM](https://www.ibm.com/docs/en/i/7.4.0?topic=programming-how-sockets-work)
- [Oracle](https://docs.oracle.com/javase/tutorial/networking/sockets/definition.html)
- [MIT](https://web.mit.edu/6.031/www/fa17/classes/24-sockets-networking/)

###### Ten64

Morgan, S. and Ismo, R. "Ten64: Text Encoded Numbers as Base 64 bit", adligo/ten64.adligo.org, GitHub Repository, March 2026, [https://github.com/adligo/ten64.adligo.org](https://github.com/adligo/ten64.adligo.org).

###### Text Media Type

- [Application Media Type rfc2046](https://www.rfc-editor.org/rfc/rfc2046.html)

###### UTF8

- [Utf8everywhere.org](http://utf8everywhere.org/)
- [Wikipedia](https://en.wikipedia.org/wiki/UTF-8)
- [Utf8.com](https://www.utf8.com/)
- [Hubspot](https://blog.hubspot.com/website/what-is-utf-8)
- [RFC](https://www.rfc-editor.org/rfc/rfc3629)

###### XML

- [World Wide Web Consortium](https://www.w3.org/TR/xml/)
- [W3schools](https://www.w3schools.com/xml/xml_whatis.asp)
- [Mozilla](https://developer.mozilla.org/en-US/docs/Web/XML/Guides/XML_introduction)
- [Amazon](https://aws.amazon.com/what-is/xml/)
- [Microsoft](https://support.microsoft.com/en-us/office/xml-for-the-uninitiated-a87d234d-4c2e-4409-9cbc-45e4eb857d44)
- [The Library of Congress](https://www.loc.gov/preservation/digital/formats/fdd/fdd000075.shtml)
- [rfc5364](https://datatracker.ietf.org/doc/html/rfc5364)

###### WebSockets

- [Mozilla](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Wikipedia](https://en.wikipedia.org/wiki/WebSocket)
- [Working Group](https://websockets.spec.whatwg.org/)
- [rfc6455](https://tools.ietf.org/html/rfc6455)
