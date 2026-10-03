# XML

`ecosystem::xml` is a pure GoML, UTF-8 XML 1.0 token reader and writer with explicit bounds. The reader accepts input chunks through `Reader::feed` and returns completed `Token` values; `finish` checks that exactly one root element closed. `read_all` is a bounded convenience wrapper. A self-closing element produces adjacent `Start` and `End` tokens. The writer consumes the same token model through `Writer::write` and returns one output chunk per token, or `write_all` combines them.

Tokens cover start/end elements, text, comments, CDATA, and processing instructions. `QName` contains prefix, local name, and resolved namespace URI. Each start element carries attributes and namespace declarations. The parser handles default and prefixed namespace scopes, the predefined `xml` prefix, duplicate expanded-attribute checks, namespace rebinding, and XML 1.0 name characters. Unprefixed attributes remain outside the default namespace, as required by [Namespaces in XML 1.0](https://www.w3.org/TR/xml-names/). The writer requires each QName's URI to agree with its in-scope binding.

Text and attributes decode the five predefined entities and decimal/hexadecimal character references. The writer escapes XML-significant characters and preserves attribute whitespace through numeric references. Literal input CR and CRLF are normalized to LF; literal attribute tab/newline/CR become spaces. Comments and CDATA remain separate tokens. Empty comments are accepted,
including at document end and across input chunks. Outside the root, only literal
XML whitespace (space, tab, CR, LF), comments, and processing instructions are
accepted; character references and other Unicode whitespace are not document
whitespace. End tags require a name immediately after `</`, with only XML
whitespace allowed between that name and `>`. UTF-8 validity and XML character ranges are checked. DTDs, custom or external entities, validation against DTD/XSD, XInclude, and non-UTF-8 encodings are deliberately unsupported. XML declarations are exposed as processing instructions; their pseudo-attributes are validated without changing parser modes. These choices avoid implicit entity expansion or network/file access. See [XML 1.0](https://www.w3.org/TR/xml/) for the grammar.

XML declarations use the XML 1.0 Fifth Edition grammar: required `version`, then
optional `encoding`, then optional `standalone`, with XML whitespace, quoted
values and no duplicate/unknown fields. A declaration must be the first markup
at byte zero, or immediately follow the initial UTF-8 BOM; preceding whitespace
is not allowed. Only UTF-8 encoding names (case-insensitive) are supported.
Version numbers matching `1.` followed by digits are processed with XML 1.0
rules, as specified by the Fifth Edition. `standalone` accepts exactly `yes` or
`no`; it does not enable DTD processing. The writer applies the same checks to
`Token::Instruction("xml", value)` before output. Extended targets such as
`xml-stylesheet` remain ordinary processing instructions. Hexadecimal character
references require the lowercase `x` in `&#x...;`; hexadecimal digits may use
either case. See [XML declarations](https://www.w3.org/TR/xml/#sec-prolog-dtd)
and [character references](https://www.w3.org/TR/xml/#sec-references).


```goml
use ecosystem::xml;

fn namespaced_tokens() -> Result[Vec[xml::Token], xml::Error] {
    let reader = xml::Reader::new(xml::Limits::standard())?;
    let output = reader.feed("<p:root xmlns:p=\"urn:example\">Hi &amp; bye".to_bytes().as_slice())?;
    output.extend(reader.feed("</p:root>".to_bytes().as_slice())?);
    output.extend(reader.finish()?);
    Result::Ok(output)
}
```

`Limits::standard()` caps total input and decoded text at 16 MiB, pending and individual tokens at 1 MiB, output at 16 MiB, depth at 128, attributes per element at 1,024, and emitted tokens at 1,000,000. All limits can be lowered. A caller must feed chunks that fit the pending limit; long unclosed text or markup fails rather than growing without bound. Syntax, namespace, entity, UTF-8, limit, state, and serde errors are recoverable `Error` values with a byte offset for parser failures.

The optional `Schema` uses `std::serde::Serialize` and `Deserialize` through an explicit mapping of flat struct fields to root attributes or direct child text elements. `ScalarKind` supports text, booleans, signed integers, and unsigned integers. Fields are required by default. The mapping rejects unknown, repeated, mixed and nested fields. Namespace matching uses expanded URI/local names, independent of a document's chosen prefix. Repeated children, nested structs and mixed content remain outside this schema adapter.

`schema.with_optional_fields(names)` returns a schema whose selected struct
fields map to `Option[T]`; it replaces the previous optional selection. Unknown
or duplicate names fail. `None` omits the attribute or child, and a missing XML
field decodes as `None`. `Some("")` remains a present empty value; present invalid
booleans or integers still fail. Encoding requires `Option[T]` for selected
fields; a field omitted by the serializer is also treated as absent. Other
fields remain required. This is an explicit omission mapping and does not add
`xsi:nil` handling. Schema construction copies namespace/field containers, and
optional selection copies names, so later caller mutations cannot change the
validated mapping. Existing `Schema::new` and public `Field` literals remain
source compatible.

Run `(cd ../verification && just ecosystem-test xml)` from this library repository to verify the library, example and its independent downstream verification, and cached build.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test xml)` also retains the library-specific smoke and compatibility checks.
