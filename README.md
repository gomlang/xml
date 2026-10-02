# XML

`ecosystem::xml` is a pure GoML, UTF-8 XML 1.0 token reader and writer with explicit bounds. The reader accepts input chunks through `Reader::feed` and returns completed `Token` values; `finish` checks that exactly one root element closed. `read_all` is a bounded convenience wrapper. A self-closing element produces adjacent `Start` and `End` tokens. The writer consumes the same token model through `Writer::write` and returns one output chunk per token, or `write_all` combines them.

Tokens cover start/end elements, text, comments, CDATA, and processing instructions. `QName` contains prefix, local name, and resolved namespace URI. Each start element carries attributes and namespace declarations. The parser handles default and prefixed namespace scopes, the predefined `xml` prefix, duplicate expanded-attribute checks, namespace rebinding, and XML 1.0 name characters. Unprefixed attributes remain outside the default namespace, as required by [Namespaces in XML 1.0](https://www.w3.org/TR/xml-names/). The writer requires each QName's URI to agree with its in-scope binding.

Text and attributes decode the five predefined entities and decimal/hexadecimal character references. The writer escapes XML-significant characters and preserves attribute whitespace through numeric references. Literal input CR and CRLF are normalized to LF; literal attribute tab/newline/CR become spaces. Comments and CDATA remain separate tokens. Empty comments are accepted,
including at document end and across input chunks. Outside the root, only literal
XML whitespace (space, tab, CR, LF), comments, and processing instructions are
accepted; character references and other Unicode whitespace are not document
whitespace. End tags require a name immediately after `</`, with only XML
whitespace allowed between that name and `>`. UTF-8 validity and XML character ranges are checked. DTDs, custom or external entities, validation against DTD/XSD, XInclude, and non-UTF-8 encodings are deliberately unsupported. XML declarations are exposed as processing instructions; their pseudo-attributes are not interpreted. These choices avoid implicit entity expansion or network/file access. See [XML 1.0](https://www.w3.org/TR/xml/) for the grammar.

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

The optional `Schema` uses `std::serde::Serialize` and `Deserialize` through an explicit mapping of flat struct fields to root attributes or direct child text elements. `ScalarKind` supports text, booleans, signed integers, and unsigned integers. The mapping rejects unknown, missing, repeated, mixed, and nested fields. Namespace matching uses expanded URI/local names, independent of a document's chosen prefix. It does not infer a mapping from GoML reflection or attempt general XML-to-object conversion; optional values, repeated children, nested structs, and mixed content remain outside this schema adapter.

Run `(cd ../verification && just ecosystem-test xml)` from this library repository to verify the library, example and its independent downstream verification, and cached build.

## Development and examples

Requires GoML 0.1.56 or newer. The `examples/basic/` example shares the root manifest. From the library root, run:

```sh
goml run --example basic
goml test
goml verify --timeout 300s
```

`goml test` builds the example and runs its tests. `goml verify` repeats the example checks as an independent module against an isolated registry snapshot. `(cd ../verification && just ecosystem-test xml)` also retains the library-specific smoke and compatibility checks.
