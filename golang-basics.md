Go:

What are Strings:
- In Go, a string is in effect a read-only slice of bytes
- Iterating through a string using a for loop and printing its value displays the decimal value of the byte held by the character. 
  - Decimal vs binary: https://www.quora.com/What-is-the-difference-between-binary-form-and-decimal-form
- Go uses UTF8 for all strings which means that each byte is a int32.
- A code point is any numerical value that defines the character and this is represented by one or more code units depending on the encoding.
- A rune is a code point.

Resources:
- https://medium.com/rungo/string-data-type-in-go-8af2b639478
- [Strings, bytes, runes and characters in Go](https://blog.golang.org/strings)


Go modules

Summary:
go mod init creates a new module, initializing the go.mod file that describes it.
go build, go test, and other package-building commands add new dependencies to go.mod as needed.
go list -m all prints the current module’s dependencies.
go get changes the required version of a dependency (or adds a new dependency).
go mod tidy removes unused dependencies.	

Resources:
- [modules](https://blog.golang.org/using-go-modules)

Logging and Errors:
Log errors that are not program threatening.
Return an error if you can’t continue or it will lead to data loss or corruption.
E.g. See sanitize.go





Ruby code:

With the current implementation, it is required that records are all on one line. 
- Multiline fields are not valid in our importer because of the sort + unique operation required for diffing. The separate
parts of the lines get broken.
- A csv_checksum is generated using md5sum on the line. Having a multiline checksum is doable but is less simple. 
- Ruby CSV library and SmarterCsv cannot read a line that has a mismatched quote even with liberal parsing. It raises an error.

Today solution:
iterate over the csv fields and remove new lines.

Future solution:
Get away from sorting and maybe uniquing.

All requirements:
1. All records are on a single line
1. No non UTF8 characters


To do:
Test all edges cases and compare:
BOM: https://medium.freecodecamp.org/a-quick-tale-about-feff-the-invisible-character-cd25cd4630e7
