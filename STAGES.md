# GAL Compiler Pipeline Stages

This document describes the compiler pipeline for a conforming GAL implementation. Each stage is described with its inputs, outputs, relevant specification sections, and relevant semantic rules from SEMANTICS.md. Stages are ordered as they would execute in a single-pass compiler.

---

## Stage 1: Lexing

**Input:** Raw source text of a translation unit.
**Output:** A flat stream of tokens with type and source location.

### What to produce
For each token, record:
- Token kind (keyword, identifier, integer literal, float literal, char literal, string literal, raw string literal, operator, punctuator)
- Raw text value
- Source location (file, line, column)

### Rules
- Skip whitespace and comments (`//` to end of line, `/*` to `*/`).
- Comments shall not be nested.
- Match keywords before identifiers — if a sequence matches both, it is a keyword.
- String literals: delimited by `"`, process escape sequences `\\` `\"` `\'` `\0`.
- Raw string literals: delimited by `"""` on both sides, no escape processing, `"""` shall not appear inside.
- Integer literals: decimal, `0x` hex, `0b` binary. Octal is not supported.
- The `_` token is a punctuator, not an identifier.
- The `#` token is a punctuator.
- `*` as visibility modifier is the same token as `*` as multiply — context is resolved in later stages.

### Spec sections
- §4.1 Source File [lex.source]
- §4.2 Comments [lex.comment]
- §4.3 Tokens [lex.token]
- §4.4 Keywords [lex.key]
- §4.5 Identifiers [lex.name]
- §4.6 Literals [lex.literal]
- §4.7 Operators and Punctuators [lex.operators]

---

## Stage 2: Preprocessing — Include and Import Resolution

**Input:** Token stream.
**Output:** Expanded token stream with all `include` directives replaced and all `import` directives resolved.

### What to do
For each `include "file"` directive:
1. Locate the file (implementation-defined search path).
2. Lex the file.
3. Replace the directive with the resulting token stream.
4. All declarations from the included file are native to the including TU.

For each `import "file"` directive:
1. Locate and fully process the named TU (all stages through name registration).
2. Make all its declarations available in the current TU's symbol table.
3. Mark non-`*` declarations as non-re-exportable.
4. If `import*`, mark `*` declarations as re-exportable from the current TU.

### Rules
- `include` is textual; declarations have no foreign boundary.
- `import` creates a boundary; non-`*` names are usable but not re-exported.
- If two imports contribute the same name at the same scope level, the program is ill-formed.

### Spec sections
- §23.1 include [dcl.import.include]
- §23.2 import [dcl.import.import]

### Semantics reference
- SEMANTICS.md §10 Import and Include Semantics

---

## Stage 3: Parsing

**Input:** Expanded token stream.
**Output:** Abstract Syntax Tree (AST).

### What to produce
An AST node for every construct in GRAMMAR.md. Key node types:

- **Declarations:** variable, type, function, forward, alias
- **Types:** fundamental, pointer, reference, value, array, tuple, object, enum
- **Expressions:** literal, identifier, binary-op, unary-op, call, subscript, field-access, cast, type-constructor, variant-construction, tuple, lambda, mixin
- **Statements:** compound, if/elif/else, while, for, case, label, goto, return, break, continue, asm

### Rules
- The grammar is in GRAMMAR.md. Every production maps to an AST node.
- `_` is context-sensitive: resolve its role (wildcard, discard, outer-scope prefix, array size) during parsing based on syntactic position.
- `|` is context-sensitive: in a type-annotation position it is a union type constructor (currently removed — treat as ill-formed); in an expression position it is bitwise OR.
- `..` is only valid inside `[ ]` array literals and `case` label positions. Reject elsewhere.
- Generic parameter lists `<` `>` are attached to function declarations and type declarations only.
- Pragma `[. .]` is attached to the immediately following declaration identifier.
- Template function bodies are stored as raw token sequences, not fully parsed, since they are expanded textually at call sites.

### Disambiguation cases
- `x * y` vs `ptr T`: resolved by context — after `:` or in a type position, `*` is not multiply.
- `f(x)` vs `T(x)`: resolved in type-checking stage — parse both as call expressions.
- `x.f()` vs field access: parse as member call; resolve in name resolution.

### Spec sections
- All of §4 Lexical Conventions [lex]
- All of §5 Types [basic.types]
- §7 Declarations [dcl]
- §8 Wildcard and Auto [dcl.wild]
- §10 Tuples [basic.tuple]
- §11 Arrays and Ranges [basic.array]
- §12 Pointers, References, Values [basic.compound]
- §13 Objects [class]
- §14 Enumerations [dcl.enum]
- §15 Functions and Generics [dcl.fct]
- §16 Templates [dcl.template]
- §18 Control Flow [stmt]
- §19 Labels and Goto [stmt.label]
- §20 Pattern Matching [stmt.case]
- §21 Specialized Symbols [over]
- §22 Scopes and Aliases [basic.scope]
- §24 Casts [expr.cast]
- §25 Function Call Syntax [over.call]
- §27 Pragmas [dcl.pragma]
- §28 Inline Assembly [stmt.asm]

### Grammar reference
- GRAMMAR.md — all productions

---

## Stage 4: Name Registration (Symbol Table Population)

**Input:** AST.
**Output:** Symbol table with all declared names mapped to their AST nodes and scopes.

### What to do
Walk the AST in declaration order and register every declared name:
- Variable declarations
- Type declarations
- Function declarations (including forward declarations)
- Enum variants
- Object fields
- Generic parameters (per instantiation)
- Labels (goto targets)
- Alias declarations

For each name, record:
- Identifier string
- Kind (variable, type, function, field, variant, label, alias, generic parameter)
- Scope (block, function, TU)
- Declaration keyword (`const`, `let`, `var`) and visibility (`*` or not)
- AST node reference

### Rules
- Two names at the same scope level that do not form an overload set are ill-formed.
- Forward declarations introduce a name that is marked as incomplete until the full definition is registered.
- `import` names are registered at TU scope; non-`*` ones are flagged as non-re-exportable.
- Labels are registered at TU scope (no two labels in a TU may share a name).

### Spec sections
- §31 Name Lookup [basic.lookup]
- §7.3 Forward Declarations [dcl.fwd]

### Semantics reference
- SEMANTICS.md §1 Name Resolution
- SEMANTICS.md §3 Scope Rules

---

## Stage 5: Type Checking and Semantic Analysis

**Input:** AST + symbol table.
**Output:** Type-annotated AST; all expressions have a resolved type.

### What to do
Walk the AST and for each expression and statement:
1. Resolve the type of every expression.
2. Check assignment compatibility.
3. Check mutability constraints.
4. Apply promotion rules to arithmetic expressions.
5. Verify lvalue/rvalue requirements (`addrof`, references, assignments).
6. Verify constant expression requirements in `const` contexts.
7. Instantiate generic functions per call site.
8. Expand template functions textually at call sites (re-enter stages 3–5 for the expanded body).
9. Check `case` arm types against the case operand type.
10. Verify variant construction and pattern types.
11. Check object construction (named args, required fields, types).
12. Verify `fieldof` and `hasfield` argument validity.

### Rules
- Type deduction: if no annotation, infer from initializer.
- Promotion: same signedness family only; `bool` and `char` require explicit conversion.
- `const` bindings: no runtime address, not writable, not rebindable.
- References: initialized at declaration, never rebound.
- Generic instantiation: substitute and re-type-check; errors reported at call site.
- Template expansion: textual substitution at call site, then re-parse and re-type-check.
- `fieldof<T>(i)`: `i` must be a constant expression; result type is the type of field `i`.
- `hasfield<T>(name)`: `name` must be a constant expression of type `[char, N]`.

### Spec sections
- §6 Type Qualifiers [basic.type.qualifier]
- §9 void [basic.void]
- §13.6 Field Initializers [class.init]
- §14 Enumerations [dcl.enum]
- §15.4 Generics [dcl.fct.generic]
- §16 Templates [dcl.template]
- §25 Function Call Syntax / UFCS [over.call]
- §30 Value Categories [expr.cat]
- §31 Name Lookup [basic.lookup]
- §32 Expressions [expr.types]

### Semantics reference
- SEMANTICS.md §2 Type Checking
- SEMANTICS.md §4 Lvalue and Rvalue Rules
- SEMANTICS.md §5 Compile-Time Evaluation
- SEMANTICS.md §6 Pattern Matching Rules
- SEMANTICS.md §7 Function Rules
- SEMANTICS.md §8 Object Rules
- SEMANTICS.md §9 Operator Overloading

---

## Stage 6: Compile-Time Evaluation

**Input:** Type-annotated AST.
**Output:** AST with all `const` declarations and `const if`/`for`/`while` bodies fully evaluated and replaced with their results. `mixin` calls executed and their output re-parsed and re-type-checked.

### What to do
1. Evaluate all `const` declarations in order.
2. Evaluate `const if` conditions; retain only the taken branch in the AST; discard the other.
3. Unroll `const for` and `const while` into their constituent iterations.
4. Execute all `mixin` calls; parse the resulting string as GAL tokens; re-enter stages 3–5 for the inserted code.
5. Unroll `template for` and `template while` textually.
6. Evaluate all built-in compile-time functions: `sizeof`, `alignof`, `offsetof`, `lenof`, `fieldof`, `typeof`, `hasfield`, `identof`, `stringof`.

### Rules
- `asm`, `goto`, `label`, `addrof` are ill-formed in compile-time contexts.
- Pointer arithmetic is ill-formed in compile-time contexts unless the pointer was created in the same `const` context.
- `mixin` output must be syntactically valid at the insertion point.
- After `mixin` insertion, name registration and type checking must be re-run for the inserted declarations.

### Spec sections
- §3.2 Evaluation Contexts [intro.eval]
- §17 Mixins [dcl.mixin]
- §18.2 Selection Statements — `const if` [stmt.if]
- §18.3 Iteration Statements — `const for/while` [stmt.iter]
- §29 Built-in Functions [over.reflect]
- §32.2 Constant Expressions [expr.types.const]

### Semantics reference
- SEMANTICS.md §5 Compile-Time Evaluation

---

## Stage 7: Code Generation

**Input:** Fully evaluated, type-annotated AST.
**Output:** Target assembly or object code.

### What to produce
For each construct, emit the appropriate target code:

- **Variable declarations:** allocate storage per MEMORY_LAYOUT.md; emit initializer code.
- **Function declarations:** emit function prologue/epilogue per calling convention; emit body.
- **Assignments:** emit load/store sequences; respect mutability (already checked).
- **Arithmetic expressions:** emit typed arithmetic instructions; operands already promoted.
- **Pointer subscript `p[N]`:** emit `p + N * sizeof<T>()` address computation and dereference.
- **Object construction:** allocate storage; evaluate field initializers in declaration order; store each field at its computed offset per MEMORY_LAYOUT.md.
- **Enum variant construction `E::V(expr)`:** store discriminant value for `V` at offset 0; store `expr` result at payload offset.
- **`case` statement:** emit a computed-goto dispatch on the case operand; for enum operands, dispatch on the discriminant field.
- **`for` loop:** evaluate iterable once; emit loop over element count.
- **`asm` statement:** emit the assembly string literally with operand bindings in GAS extended inline syntax.
- **Pragmas:** apply code generation hints (`[.inline.]`, `[.align.]`, `[.section.]`, `[.volatile.]`, etc.).

### Memory layout rules
All size, alignment, offset, and padding decisions are governed by MEMORY_LAYOUT.md.

### Calling conventions
- Default calling convention is implementation-defined.
- `[.convention("name").]` overrides the calling convention for a specific function.
- `[.asmStackframe: off.]` suppresses the compiler-generated stack frame.
- `[.noreturn.]` suppresses return code generation.
- UFCS calls are emitted identically to regular calls with the receiver as the first argument.

### Visibility and linkage
- Names marked `*` are exported in the output binary.
- Names without `*` have internal linkage.
- `[.export("name").]` overrides the exported symbol name.
- `[.import("name").]` imports a symbol from an external binary by the given name.
- `[.section("name").]` places the symbol in the named output section.
- `[.static.]` fields are emitted at TU scope, not within any object instance.

### Spec sections
- §12 Pointers, References, Values [basic.compound]
- §13 Objects [class] + §13.6 Field Initializers [class.init]
- §14 Enumerations [dcl.enum]
- §19 Labels and Goto [stmt.label]
- §20 Pattern Matching [stmt.case]
- §26 Operator Precedence [expr.prec]
- §27 Pragmas [dcl.pragma]
- §28 Inline Assembly [stmt.asm]
- §32.3 Integer Arithmetic [expr.types.int]

### Memory layout reference
- MEMORY_LAYOUT.md — all sections
