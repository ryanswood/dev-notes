Go:

Language resources:
- [Quick start guide](https://medium.freecodecamp.org/learning-go-from-zero-to-hero-d2a3223b3d86)
- [Language spec](https://golang.org/ref/spec)
- [Effective Go](https://golang.org/doc/effective_go.html)


## Strings:
- Immutable. Uses double quotes.
- In Go, a string is in effect a read-only slice of bytes
- Iterating through a string using a for loop and printing its value displays the decimal value of the byte held by the character. 
  - Decimal vs binary: https://www.quora.com/What-is-the-difference-between-binary-form-and-decimal-form
- Go uses UTF8 for all strings which means that each byte is a int32.
- A code point is any numerical value that defines the character and this is represented by one or more code units depending on the encoding.
- A rune is a code point.
- There is not Ruby style interpolation due to static types: https://medium.com/@channaly/string-interpolation-in-golang-ecb3bcb2d600

Example: "a"

Resources:
- https://medium.com/rungo/string-data-type-in-go-8af2b639478
- [Strings, bytes, runes and characters in Go](https://blog.golang.org/strings)

## Rune literals:
A rune literal represents a rune constant, an integer value identifying a Unicode code point. A rune literal is expressed as one or more characters enclosed in single quotes, as in 'x' or '\n'.

Example: 'a'

Resources:
- https://golang.org/ref/spec#Rune_literals

## String literals
A string literal represents a string constant obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.

- Types
  - Raw string literals are character sequences between back quotes, as in `foo`. Within the quotes, any character may appear except back quote. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes have no special meaning and the string may contain newlines.
  - Interpreted string literals are character sequences between double quotes, as in "bar". Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in rune literals (except that \' is illegal and \" is legal), with the same restrictions. The three-digit octal (\nnn) and two-digit hexadecimal (\xnn) escapes represent individual bytes of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual characters. Thus inside a string literal \377 and \xFF represent a single byte of value 0xFF=255, while ÿ, \u00FF, \U000000FF and \xc3\xbf represent the two bytes 0xc3 0xbf of the UTF-8 encoding of character U+00FF.
  
Examples:
- `abc` == "abc"
```
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

Resources:
- https://golang.org/ref/spec#String_literals

Printing + Interpolation + Concatenation

- https://yourbasic.org/golang/fmt-printf-reference-cheat-sheet/
- https://medium.com/@channaly/string-interpolation-in-golang-ecb3bcb2d600
- https://www.calhoun.io/6-tips-for-using-strings-in-go/


## Go modules

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

## Testing
Go includes a test runner in the go toolchain. 

Resources:
- https://blog.alexellis.io/golang-writing-unit-tests/

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
