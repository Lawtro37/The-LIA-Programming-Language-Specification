<div style="display:flex;align-items:center;gap:16px;">
  <img src="logo.png" width="200" alt="LIA Logo">
  <div>
    <h1>The LIA Programming  Language v1.0.0 Specification [DRAFT]</h1>
    Revision: 1<br>
    Release Date: DD/MM/YYYY<br>
    Author: Lawton Kelly
  </div>
</div>

---

This document describes the core language specifications and not particularly the current state of LIA-lang. This is simpley the target of what LIA-lang should be.

**THIS DOCUMENT IS INCOMPLETE:** more content is planed to be added to this document.

## Introduction
This specification defines the normative syntax, semantics, and external interface of the LIA programming language. Its goal is to provide a precise, implementation‑agnostic contract for:
- Lexical elements (tokens, literals, keywords)
- Grammar and syntactic forms
- Type system and coercion model
- Runtime constructs (functions, objects, arrays)
- Memory and lifetime directives (`free`)
- Module and external library importing
- Native interoperability surface

Scope: Only the core language and its required runtime behaviors.  
Non‑scope: Tooling, build systems, optimization strategies, debugging facilities, editor integration, and prospective future extensions.

Conformance: An implementation is conforming if it:
1. Accepts all grammatically valid programs defined herein.
2. Rejects (with a diagnostic) any lexically or syntactically invalid input.
3. Preserves the specified evaluation order and side‑effects.
4. Implements the defined type and value categories (including `null` and `void` semantics).

Undefined Behavior: Any construct not explicitly covered (e.g. accessing non‑existent object fields) is undefined and may terminate execution or produce implementation‑specific results.

Versioning: This draft targets v1.0.0. Backward incompatible changes after ratification require a major version increment.

Terminology:
- “Must” / “Shall”: mandatory.
- “Should”: recommended.
- “May”: optional.
- “Undefined”: intentionally unspecified—implementations are not required to diagnose.
- "LIA-lang: the LIA Programming  language

All examples are illustrative; if an example conflicts with a formal rule, the rule governs.

## 1. Lexical Structure
Character set: ASCII (string contents treated as raw bytes)  
Whitespace: space, tab, CR, LF (insignificant outside string literals)  
Comments: `//` to end of line  

### 1.1 Identifiers
Pattern: `[A-Za-z_][A-Za-z0-9_]*`  
Case-sensitive. Reserved keywords excluded.

### 1.2 Keywords
`function, var, const, return, if, else, while, free, import, true, false, null, void, bool, int, float, char, string`

#### Aliases
`function`: `fn, fun, func, def`  
`var`: `let, local`  
`const`: `final` (the word `const` itself)  
`free`: `delete, pop`  
`null`: `nul`  
`bool`: `boolean`

### 1.3 Literals
- Integer: decimal digits (`123`). Parsed as signed 32‑bit unless a cast is applied.
- Float: digits with a decimal point (`1.0`, `0.5`).
- String: `"..."` (no escape sequences; backslashes are literal).
- Boolean: `true` / `false`.
- Null: `null` (internally `Type::Null`).
- Object literal: `{ <field_list> }`.
- Array literal: `[<type>? element (, element)*]` (optional leading inline type token).

### 1.4 Operators
- Arithmetic: `+ - * /`
- Comparison: `==`
- Assignment: `=`
- Unary: prefix `+ -`
- Index: `ident[expression]`
- Member access: grammar placeholder exists but is not part of the active syntax; accessing object fields directly is not defined outside literal construction.

### 1.5 Delimiters
`() [] { } , :`

## 2. Types
Primitive core:
- `int<S>` / `uint<S>` (signed / unsigned integers by size)
- `float<S>`
- `char<S>` (signedness encoded similarly to integers)
- `string`
- `bool`
- `void` (function return type only)
- `null`
- `any` (internal fallback / inference default)

`S` = storage size in bytes; default size when omitted is 32 bits.

#### Sub‑type Aliases
Default integer width: 32 bits (alias `int` = `int32`).

Integers (signed / unsigned):
- 8:  `int8,  i8,  byte` / `uint8,  u8,  ubyte`
- 16: `int16, i16, short` / `uint16, u16, ushort`
- 32: `int32, i32` / `uint32, u32` (alias `int` = `int32`)
- 64: `int64, i64` / `uint64, u64`

Floats:
- 32: `float, float32, f32, number, num`
- 64: `float64, f64, double, real`

Chars (signed / unsigned):
- 8:  `char8` / `uchar8`
- 16: `char16` / `uchar16`
- 32: `char, char32` / `uchar, uchar32`
- 64: `char64` / `uchar64`

Strings: `string, str, text`

Composite:
- Arrays: dynamic length; literal syntax `[...]`.
- Objects: anonymous record literal `{ type name: value, ... }`.
- Functions: first‑class by identifier (no closures).

## 3. Variables
Syntax:
`var <type> <name> (= expression)?`
`var <type> <name>[index]?` (array element assignment after declaration uses assignment)
`const` modifier is parsed; enforcement of immutability is nominal.

Uninitialized variables get `null`.

## 4. Statements
- Variable declaration
- Expression statement
- `if (expr) { block } (else { block })?`
- `while (expr) { block }`
- `return expression`
- `free(identifier (, identifier)*)`
- `import "path.lia" | "lib.llb"`

Blocks: `{ statement* }` produce an internal `Program` node.

## 5. Functions
Syntax:
`function <return_type> <identifier>( <param_list>? ) { block }`

Parameters:
`<type> <identifier>` separated by commas.

Return:
`return <expression>` (void functions may return without an expression producing `null`).

No overloading or default parameters.

## 6. Objects
Literal:
`{ <type> <fieldName>: <expression>, ... }`
Semantics: constructs `AST::Object(Vec<(Type,String,AST)>)`.

## 7. Arrays
Two forms:
- Indexed identifiers: `arr[i]`
- Literals: `[ <optionalLeadingType> elem1, elem2, ... ]`
Stored as `AST::Array(Type, Vec<AST>)`; element type is either explicit leading type token or inferred as `Any`.

## 8. Casting
Prefix type token followed by a parenthesized expression: `int32(expr)` is parsed as a call‑style cast (`AST::Call("int32", [expr])`).

## 9. Import System
`import "relative/or/absolute/path.lia"`  
- Loads and tokenizes file; AST is inlined (recursive parse).  
Optional (commented / experimental) dynamic library form: `import "library.llb"`.

Search path: relative to current working directory. No cycle detection.

## 10. Memory Management
Manual release hook:
`free(a, b, c)` builds `AST::Free(Vec<Identifier>)`. Runtime decides how to dispose / release.

## 11. Runtime / Execution Model
- Tree‑walk interpreter
- Optimizer stage
- Serialization: `.ast` (serialized AST)
- Native extension (experimental): dynamic function registration via DLL / `.llb`.

## 12. Error Handling
Current: panic on parse errors (immediate abort).

## 13. Grammar (EBNF Approximation)
```
program          ::= statement*
statement        ::= function_decl
				   | var_decl
				   | if_stmt
				   | while_stmt
				   | return_stmt
				   | free_stmt
				   | import_stmt
				   | expression
function_decl    ::= "function" type identifier "(" param_list? ")" block
param_list       ::= param ("," param)*
param            ::= type identifier
var_decl         ::= "var" ("const")? type identifier ("=" expression)?
if_stmt          ::= "if" "(" expression ")" block ("else" block)?
while_stmt       ::= "while" "(" expression ")" block
return_stmt      ::= "return" expression
free_stmt        ::= "free" "(" ident_or_index ("," ident_or_index)* ")"
import_stmt      ::= "import" string_literal
block            ::= "{" statement* "}"
expression       ::= assignment
assignment       ::= binary ("=" assignment)?
binary           ::= unary (op unary)*        # op in precedence order (* /, + -, ==)
unary            ::= ("+" | "-" | cast_prefix) unary | primary
cast_prefix      ::= type "("
primary          ::= number
				   | string_literal
				   | "true" | "false" | "null"
				   | identifier call_or_index?
				   | array_literal
				   | object_literal
				   | "(" expression ")"
call_or_index    ::= "(" arg_list? ")" | "[" expression "]"
arg_list         ::= expression ("," expression)*
array_literal    ::= "[" type? (expression ("," expression)*)? "]"
object_literal   ::= "{" object_field ("," object_field)* "}"
object_field     ::= type identifier ":" expression
ident_or_index   ::= identifier ("[" expression "]")?
type             ::= "int" size? | "uint" size? | "float" size? | "char" size? | "string" | "bool" | "void" | "any"
size             ::= integer_literal
```

## 14. Operator Precedence (High → Low)
1. Unary (`+ - (cast)`)
2. Multiplicative (`* /`)
3. Additive (`+ -`)
4. Equality (`==`)
5. Assignment (`=` right-associative)

## 15. Native Interop
Dynamic library import (if enabled) expects at minimum: `extern "C" fn register_functions()`. A callback variant `register_functions_with_callback` may also be recognized.

## 16. Compliance Notes
Where implementation and this document differ (e.g. disabled member access), the source code is authoritative until harmonised.

---
Status: Draft baseline for v1.0.0 core.
