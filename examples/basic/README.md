# XML example

This example uses the library in this repository. It exercises bounded token parsing and writing plus an explicit serde schema for a flat struct.

Run `(cd ../../../verification && just ecosystem-test xml)` from this example directory.

This example shares the library root manifest and development dependencies. Run `goml verify --example basic` to build and test it as an independent downstream module.
