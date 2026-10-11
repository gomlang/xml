# XML example

This example uses the library in this repository. It exercises bounded token parsing and writing plus an explicit serde schema for a flat struct.

Run `(cd ../../../workflows && just ecosystem-test xml)` from this example directory.

This example shares the library root manifest and development dependencies. From the library root, run `goml test` to build and test it, or `(cd ../workflows && just ecosystem-test xml)` to check it against an isolated registry snapshot.
