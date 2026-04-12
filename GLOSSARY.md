# GAL Glossary

All terms used in the GAL specification, cross-referenced to their defining section.

---

**addrof** — a built-in function that yields the address of an lvalue as a pointer. Cannot be overloaded. The operand shall be an lvalue and shall not be a `const` binding. [basic.compound.addr, over.reflect.addr]

**array** — a fixed-length, contiguous sequence of elements of a single type. Not implicitly null-terminated. [basic.array.general]

**auto** — a type specifier indicating an anonymous but concrete type. Assembly is generated for `auto`-typed declarations. Lambdas have type `auto`. [dcl.wild.auto, dcl.fct.lambda]

**binding** — the association of an identifier with a value or storage location, established by a declaration. [intro.defs]

**break** — a statement that exits the innermost enclosing loop, or exits a `case` arm early. [stmt.general, stmt.case]

**case statement** — a scoped computed-goto construct that evaluates an expression and transfers control to a matching label. Supports integer, range, enum variant, and wildcard labels. Fall-through does not occur. [stmt.case]

**compile-time context** — an evaluation context in which all expressions are evaluated by the compiler prior to code generation. [intro.defs, intro.eval]

**compound statement** — a `{}`-delimited sequence of statements that introduces a new scope. [stmt.general, basic.scope.general]

**const** — a declaration keyword that requires compile-time evaluation. The binding cannot be rebound and has no runtime address. [dcl.var]

**constant expression** — an expression fully evaluable at compile time. [expr.types.const]

**declaration keyword** — one of `const`, `let`, or `var`. Controls rebindability of a binding and sets the default mutability qualifier for its type. [intro.defs, dcl.var]

**discriminant** — an implicit integer tag assigned by the compiler to each variant of an enumeration, in declaration order beginning at zero. Used by `case` to identify the active variant at runtime. [dcl.enum.general]

**element binding** — the variable bound to each element in a `for` loop iteration. [stmt.iter]

**enum** — a discriminated type whose variants each carry either a fixed value or a type. Every variant shall be assigned a value or type. [dcl.enum.general]

**eval-prefix** — `const` or `template` applied to a function or loop declaration to control evaluation timing or textual expansion. [dcl.fct.general, stmt.iter]

**field** — a named member of an object type. Fields are parameter bindings. [class.general]

**fieldof** — a built-in function that yields the field of an object type at a given constant index. Fields are ordered base-first. [over.reflect.object]

**for overload** — a function named `` `for` `` that makes a type iterable by returning a `(T, usize)` tuple. [over.for]

**forward declaration** — a declaration that introduces a name before its full definition. Required for mutual recursion and single-pass compilation. A forward-declared type is incomplete and may only be used as a pointer target. [dcl.fwd]

**generic parameter** — a compile-time type or value placeholder in a function declaration, implicitly `const`, resolved by substitution at each call site. [dcl.fct.generic]

**hasfield** — a built-in function that yields `true` if a type has a field with a given name, regardless of visibility. Evaluated at compile time. [over.reflect.type]

**identifier** — a sequence of letters, digits, and underscores not beginning with a digit and not a keyword. [lex.name]

**ill-formed** — a program that violates a normative "shall" constraint. The compiler shall reject it, possibly with a diagnostic. [intro.defs]

**imut** — a type qualifier specifying that a value shall not be modified after initialization. [basic.type.qualifier]

**implementation-defined behavior** — behavior not specified by this document but defined and documented by each conforming implementation. [intro.defs, intro.impldef]

**import** — a directive that makes all declarations of another translation unit available in the current translation unit. Non-`*` symbols are local to the importer and shall not be re-exported. [dcl.import.import]

**include** — a directive that textually inserts another source file at the point of inclusion. All declarations become native to the including translation unit. [dcl.import.include]

**incomplete type** — a type that has been forward-declared but not yet fully defined. May only be used as a pointer target. [dcl.fwd]

**inheritance list** — the comma-separated list of parent object types in an object declaration. [class.inherit]

**initializer** — the `= expression` part of a variable declaration or field. [dcl.var, class.init]

**label** — a named location in the instruction stream, target of `goto` or `case`. [stmt.label]

**lambda** — an anonymous function expression of type `auto`. [dcl.fct.lambda]

**lenof** — a built-in function that yields the element count of an array or the total field count of an object or enum type, including inherited fields, in base-first order. [over.reflect.len]

**let** — a declaration keyword that makes the binding immutable (cannot be rebound). The value's mutability is controlled independently by the type qualifier. [dcl.var]

**lvalue** — an expression that denotes a named, addressable storage location. [expr.cat.lvalue]

**mixin** — a built-in that converts a string expression to GAL tokens and inserts them at the point of call. The resulting token sequence shall be syntactically valid. [dcl.mixin, over.reflect.codegen]

**mut** — a type qualifier specifying that a value or pointed-to data may be modified. [basic.type.qualifier]

**object** — a record type containing named fields, supporting inheritance and static members. [class.general]

**operator overload** — a function whose name is an operator symbol enclosed in backticks, invoked through operator syntax. [over.general]

**parameter binding** — a variable binding introduced by a function parameter, object field, or iteration statement. Defaults to `let` if no declaration keyword is specified. [intro.defs]

**pragma** — a compile-time directive enclosed in `[. .]` that affects code generation, linkage, or diagnostics without changing program semantics. [dcl.pragma.general]

**ptr** — a type constructor for raw pointer types. Pointed-to values are accessed via subscript `p[0]`. [basic.compound.ptr]

**qualified variant** — a variant name qualified by its enclosing enum type, of the form `EnumType::VariantName`. [dcl.enum.type]

**range expression** — a syntactic construct `N1 .. N2` valid only inside `[ ]` array literals and `case` label positions. Not a type or storable value. [basic.array.range]

**ref** — a type constructor for reference types, which alias existing objects. A reference shall be initialized at declaration and cannot be rebound. [basic.compound.ref]

**rvalue** — an expression that does not denote an addressable location. [expr.cat.rvalue]

**shall** — a normative requirement. Violation renders a program ill-formed. [intro.defs]

**specialized symbol** — an operator or built-in construct invoked through syntax rather than standard function call notation. [over.general]

**template** — a keyword that causes textual expansion of a function or loop body at the point of use, at compile time. Distinct from generics. [dcl.template.general]

**template function** — a function prefixed with `template` whose body is textually embedded at every call site. Shall not declare a return type. [dcl.template.fn]

**this** — an implicitly available pointer to the enclosing object instance, valid only within an object body. [class.this]

**This** — an implicitly available type alias for the enclosing object type, valid only within an object body. [class.this]

**translation unit** — a single source file and all files textually included into it. [intro.defs]

**tuple** — an ordered, fixed-length, heterogeneous sequence of values accessed by destructuring. [basic.tuple.general]

**type annotation** — the `: type` part of a declaration. [dcl.var]

**type qualifier** — `mut` or `imut`; describes the mutability of a value independently of its binding rebindability. [basic.type.qualifier]

**type variant** — an enum variant whose assigned value is a type rather than an expression. A type variant wraps a value of the given type and may be pattern-matched with a binding. [dcl.enum.type]

**type-as-constructor** — using a type name as a function to perform non-reinterpret conversion. Distinct from `cast`. [dcl.fct.typecast]

**UFCS** — Uniform Function Call Syntax; calling a free function as if it were a method on its first argument. `x.f(y)` is equivalent to `f(x, y)`. [over.call]

**undefined behavior** — behavior for which the specification imposes no requirements. Programs exhibiting undefined behavior are not valid GAL programs. [intro.defs, intro.undef]

**val** — a type constructor for value types that bind to rvalues. [basic.compound.val]

**value variant** — an enum variant whose assigned value is an expression. Accessing the variant via `::` yields the assigned value directly. [dcl.enum.value]

**var** — a declaration keyword that makes the binding mutable (may be rebound to a compatible type). [dcl.var]

**variant** — a named member of an enumeration, holding either a fixed expression value or a wrapped value of a type. [dcl.enum.general]

**variant construction** — the syntax `EnumType::Variant(expr)` that wraps a value into a type variant. [dcl.enum.type]

**variant pattern** — a `case` label expression matching an enum variant, optionally binding the inner value of a type variant. [stmt.case]

**visibility modifier** — the `*` symbol applied to a declaration, marking it as exported from its translation unit or scope. [lex.vis]

**void** — a first-class type representing the absence of a value. May appear as a generic argument. [basic.void]

**wildcard** — the `_` symbol, used to discard values, deduce sizes, omit bindings, access outer scopes via `_::`, or as a default arm in `case`. [dcl.wild.underscore]
