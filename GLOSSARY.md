# GAL Glossary

All terms used in the GAL specification, cross-referenced to their defining section.

---

**active type** — the type resolved at compile time for a union binding, fixed for the lifetime of the binding. [basic.union]

**addr** — a built-in specialized symbol that yields the address of an lvalue as a pointer. Cannot be overloaded. [basic.compound.addr]

**array** — a fixed-length, contiguous sequence of elements of a single type, carrying a `.len` member. [basic.array.general]

**auto** — a type specifier indicating an anonymous but concrete type. ASM is generated for `auto`-typed declarations. [dcl.wild.auto]

**binding** — the association of an identifier with a value or storage location, established by a declaration. [intro.defs]

**case statement** — a scoped computed-goto construct that evaluates an expression and transfers control to a matching label. [stmt.case]

**compile-time context** — an evaluation context in which all expressions are evaluated by the compiler prior to code generation. [intro.defs, intro.eval]

**compound statement** — a `{}`-delimited sequence of statements that introduces a new scope. [stmt.general, basic.scope.general]

**const** — a declaration keyword that requires compile-time evaluation. The binding cannot be rebound. [dcl.var]

**constant expression** — an expression fully evaluable at compile time. [expr.types.const]

**declaration keyword** — one of `const`, `let`, or `var`. Controls rebindability of a binding. [intro.defs, dcl.var]

**element binding** — the variable bound to each element in a `for` loop iteration. [stmt.iter]

**enum** — a tagged union whose variants may hold values of any type. [dcl.enum]

**eval-prefix** — `const` or `template` applied to a function or loop declaration. [dcl.fct.general, stmt.iter]

**field** — a named member of an object type. [class.general]

**for overload** — a function named `` `for` `` that makes a type iterable by returning a `(T, usize)` tuple. [over.for]

**forward declaration** — an explicit declaration of a name before its full definition, required in GAL since forward references are not supported. [basic.lookup.general]

**generic parameter** — a type placeholder in a function or type declaration, resolved at each call site. [dcl.fct.generic]

**identifier** — a sequence of letters, digits, and underscores not beginning with a digit and not a keyword. [lex.name]

**ill-formed** — a program that violates a normative "shall" constraint. [intro.defs, READING.md]

**imut** — a type qualifier specifying that a value shall not be modified after initialization. [basic.type.qualifier]

**implementation-defined behavior** — behavior not specified by the document but defined and documented by each conforming implementation. [intro.defs, intro.impldef]

**import** — a directive that makes only exported (`*`) declarations of another translation unit available. [dcl.import.import]

**include** — a directive that textually inserts another source file at the point of inclusion. [dcl.import.include]

**inheritance list** — the comma-separated list of parent object types in an object declaration. [class.inherit]

**initializer** — the `= expression` part of a variable declaration. [dcl.var]

**label** — a named location in the instruction stream, target of `goto`. [stmt.label]

**lambda** — an anonymous function expression of type `auto`. [dcl.fct.lambda]

**let** — a declaration keyword that makes the binding immutable (cannot be rebound). [dcl.var]

**lvalue** — an expression that denotes a named, addressable storage location. [expr.cat.lvalue]

**mixin** — a statement that converts a string expression to GAL tokens and inserts them at the point of use. [dcl.mixin]

**mut** — a type qualifier specifying that a value or pointed-to data may be modified. [basic.type.qualifier]

**object** — a record type containing named fields. [class.general]

**operator overload** — a function whose name is an operator symbol enclosed in backticks, invoked through operator syntax. [over.general]

**pragma** — a compile-time directive enclosed in `[. .]` that affects code generation, linkage, or diagnostics without changing program semantics. [dcl.pragma.general]

**ptr** — a type constructor for raw pointer types. [basic.compound.ptr]

**range expression** — a syntactic construct `N1 .. N2` that produces an array or serves as a case label. Not a storable type. [basic.array.range]

**ref** — a type constructor for reference types, which alias existing objects. [basic.compound.ref]

**rvalue** — an expression that does not denote an addressable location. [expr.cat.rvalue]

**shall** — a normative requirement. Violation renders a program ill-formed. [intro.defs]

**specialized symbol** — an operator or built-in construct invoked through syntax rather than standard function call notation. [over.general]

**template** — a keyword that causes textual expansion of a function or loop at the point of use, at compile time. [dcl.template.general]

**template function** — a function prefixed with `template` whose body is textually embedded at every call site. [dcl.template.fn]

**this** — an implicitly available pointer to the enclosing object, valid only within an object body. [class.this]

**This** — an implicitly available type alias for the enclosing object type, valid only within an object body. [class.this]

**translation unit** — a single source file and all files textually included into it. [intro.defs]

**tuple** — an ordered, fixed-length, heterogeneous sequence of values accessed by destructuring. [basic.tuple.general]

**type annotation** — the `: type` part of a declaration. [dcl.var]

**type qualifier** — `mut` or `imut`; describes the mutability of a value independently of its binding. [basic.type.qualifier]

**type-as-constructor** — using a type name as a function to perform non-reinterpret conversion. [dcl.fct.typecast]

**UFCS** — Uniform Function Call Syntax; calling a free function as if it were a method on its first argument. [over.call]

**undefined behavior** — behavior for which the specification imposes no requirements. [intro.defs, intro.undef]

**union** — a type whose active type is one of several alternatives, resolved at compile time. [basic.union]

**val** — a type constructor for value types that bind to rvalues. [basic.compound.val]

**var** — a declaration keyword that makes the binding mutable (may be rebound). [dcl.var]

**variant** — a named member of an enumeration, holding a value of any type. [dcl.enum]

**visibility modifier** — the `*` symbol applied to a declaration, marking it as exported from its translation unit. [lex.vis]

**void** — a first-class type representing the absence of a value. [basic.void]

**wildcard** — the `_` symbol, used to discard values, deduce sizes, omit bindings, or refer to outer scopes. [dcl.wild.underscore]
