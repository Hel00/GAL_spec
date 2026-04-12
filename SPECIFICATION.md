# GAL — General Applicative Language
## Working Draft Specification

---

# Contents

1. Scope [intro.scope]
2. Terms and Definitions [intro.defs]
3. General Principles [intro.general]
4. Lexical Conventions [lex]
5. Types [basic.types]
6. Type Qualifiers [basic.type.qualifier]
7. Declarations [dcl]
8. The Wildcard and Anonymous Types [dcl.wild]
9. `void` [basic.void]
10. Tuples [basic.tuple]
11. Arrays and Ranges [basic.array]
12. Pointers, References, and Values [basic.compound]
13. Objects [class]
14. Enumerations [dcl.enum]
15. Functions and Generics [dcl.fct]
16. Templates [dcl.template]
17. Mixins [dcl.mixin]
18. Control Flow [stmt]
19. Labels and Goto [stmt.label]
20. Pattern Matching [stmt.case]
21. Specialized Symbols and Operator Overloading [over]
22. Scopes and Aliases [basic.scope]
23. Import and Include [dcl.import]
24. Casts [expr.cast]
25. Function Call Syntax [over.call]
26. Operator Precedence [expr.prec]
27. Pragmas [dcl.pragma]
28. Inline Assembly [stmt.asm]
29. Built-in Functions [over.reflect]
30. Value Categories [expr.cat]
31. Name Lookup [basic.lookup]
32. Expressions [expr.types]

---

## 1 Scope [intro.scope]

[1] This document specifies the syntax and semantics of the General Applicative Language (GAL). It does not specify the behavior of the standard library, compiler diagnostics beyond what is required, or implementation strategies.

---

## 2 Terms and Definitions [intro.defs]

[1] The following terms are used throughout this document.

**binding** — the association of an identifier with a value or storage location, established by a declaration.

**compile-time context** — an evaluation context in which all expressions are evaluated by the compiler prior to code generation.

**declaration keyword** — one of `const`, `let`, or `var`; controls the rebindability of a binding.

**parameter binding** — a variable binding introduced by a function parameter, object field, or iteration statement. Parameter bindings follow the same declaration rules as variable declarations ([dcl.var]). The declaration keyword defaults to `let` if omitted.

**translation unit** — a single source file and all files textually included into it.

**undefined behavior** — behavior for which this document imposes no requirements. Programs that exhibit undefined behavior are ill-formed, no diagnostic required.

**implementation-defined behavior** — behavior that is not specified by this document but is defined by each conforming implementation, which shall document it.

**shall** — a normative requirement. A program that violates a "shall" constraint is ill-formed.

---

## 3 General Principles [intro.general]

### 3.1 Translation [intro.translation]

[1] A GAL program consists of one or more translation units. Translation units are combined through `import` and `include` directives ([dcl.import]).

### 3.2 Evaluation Contexts [intro.eval]

[1] Expressions are evaluated either at compile time or at runtime. The keyword `const` applied to a declaration or statement forces evaluation at compile time. The statement `if const` ([stmt.if]) tests whether the current evaluation context is compile time.

[2] An expression that cannot be evaluated at compile time in a `const` context renders the program ill-formed.

### 3.3 Undefined Behavior [intro.undef]

[1] Certain constructs in this document are explicitly designated as undefined behavior. A conforming implementation may assume that undefined behavior does not occur. The consequences of undefined behavior are unpredictable.

### 3.4 Implementation-Defined Behavior [intro.impldef]

[1] The bit width of each abstract size unit (byte, word, double-word, quad-word, octa-word) is implementation-defined. Conforming implementations shall document the width of each unit for the target platform.

---

## 4 Lexical Conventions [lex]

### 4.1 Source File [lex.source]

[1] A GAL source file is a sequence of tokens. Whitespace and comments are not tokens but serve as token separators.

### 4.2 Comments [lex.comment]

[1] A line comment begins with `//` and extends to the end of the source line. A block comment begins with `/*` and ends with `*/`. Comments shall not be nested. Comments are treated as whitespace.

### 4.3 Tokens [lex.token]

[1] The tokens of GAL are: keywords, identifiers, literals, operators, and punctuators.

### 4.4 Keywords [lex.key]

[1] The following identifiers are reserved as keywords and shall not be used as user-defined identifiers:

```
addrof   alias    alignof  asm      auto     bool     break    case
cast     char     const    continue elif     else     enum     fieldof
fn       for      goto     if       imut     import   include  identof
label    lenof    let      mixin    mut      object   offsetof ptr
ref      return   sizeof   stringof template this     This     type
typeof   val      var      void
```

### 4.5 Identifiers [lex.name]

[1] An identifier is a sequence of one or more letters, digits, and underscores that does not begin with a digit and is not a keyword.

### 4.6 Literals [lex.literal]

[1] GAL supports the following kinds of literals: integer literals, floating-point literals, character literals, string literals, and boolean literals.

[2] A string literal is a sequence of characters enclosed in double quotes. A string literal has no intrinsic type; it is used as an initializer for an array of `char` or a pointer to `char` ([basic.array], [basic.compound]). String literals are not null-terminated unless the null character `\0` is explicitly present in the literal.

[3] A character literal is a single character enclosed in single quotes and has type `char`.

[4] Integer literals may be expressed in decimal, hexadecimal (prefixed `0x`), or binary (prefixed `0b`). Octal literals are not supported.

[5] The default type of an integer literal with no explicit type annotation is `intd`. The default type of a floating-point literal is `floatd`.

[6] The following escape sequences are recognized within string and character literals:

| Sequence | Meaning |
|----------|---------|
| `\\` | literal backslash |
| `\"` | literal double quote |
| `\'` | literal single quote |
| `\0` | null character |

[7] Escape sequences produce the corresponding character value; `\0` produces the character with value zero (0x00).

[8] A raw string literal is delimited by `"""` on both sides. No escape sequences are processed within a raw string literal; all characters are taken as-is including newlines. A raw string literal has no intrinsic type and follows the same usage rules as a regular string literal. It is ill-formed for the sequence `"""` to appear within a raw string literal.

[9] *Example:*
```
let a: [char, _] = "hello \"world\"";
let b: [char, _] = """
this is
a raw string
""";
```

### 4.7 Operators and Punctuators [lex.operators]

[1] The following tokens are operators and punctuators:

```
{   }   [   ]   (   )   ::  .   ->
+   -   *   /   %   =   +=  -=  *=  /=  %=  ==  !=
<   >   <=  >=  &&  ||  !   &   |   ^   ~   <<  >>
&=  |=  ^=  <<= >>=  ..  ,   :   ;   _   #
```

### 4.8 The Visibility Modifier [lex.vis]

[1] The `*` symbol, when applied to a declaration, marks the declared name as visible outside its translation unit. A name declared without `*` is private to its translation unit and shall not be accessed from another translation unit.

### 4.9 Scope Resolution [lex.scope]

[1] The `::` operator is the scope resolution operator. It is used to access members of enumerations ([dcl.enum]), named scopes, and aliases ([basic.scope]).

[2] Object members are accessed with `.`. Members of a pointer-to-object are accessed with `->`. The `::` operator shall not be used to access non-static object members. Static members are declared with `[.static.]` ([dcl.pragma.standalone]).

---

## 5 Types [basic.types]

### 5.1 General [basic.types.general]

[1] Every expression in a GAL program has a type. Types are either fundamental, compound, or user-defined.

[2] The bit width of each abstract size unit is implementation-defined ([intro.impldef]). The size designators are:

- **b** — byte
- **w** — word
- **d** — double-word
- **q** — quad-word
- **o** — octa-word

### 5.2 Integral Types [basic.types.integral]

[1] The signed integral types are `intb`, `intw`, `intd`, `intq`, and `into`. The unsigned integral types are `uintb`, `uintw`, `uintd`, `uintq`, and `uinto`.

[2] `usize` is an unsigned integer type of implementation-defined width sufficient to represent the size of any object. `ssize` is its signed counterpart.

### 5.3 Floating-Point Types [basic.types.float]

[1] The floating-point types are `floatw`, `floatd`, `floatq`, and `floato`.

### 5.4 The Boolean Type [basic.types.bool]

[1] The type `bool` has exactly two values: `true` and `false`.

### 5.5 The Character Type [basic.types.char]

[1] The type `char` represents a single character. Its width is implementation-defined.

### 5.6 The `type` Type [basic.types.type]

[1] `type` is a meta-type used to denote types in generic parameter declarations and type declarations.

[2] `type` does not participate in type expressions ([basic.types.syntax]).

[3] *Example:*
```
type MyInt = intq;
```

### 5.7 Type Syntax [basic.types.syntax]

[1] *Syntax:*

> *type:*
> &nbsp;&nbsp; *fundamental-type*
> &nbsp;&nbsp; *pointer-type*
> &nbsp;&nbsp; *reference-type*
> &nbsp;&nbsp; *value-type*
> &nbsp;&nbsp; *array-type*
> &nbsp;&nbsp; *tuple-type*
> &nbsp;&nbsp; *object-type*
> &nbsp;&nbsp; *enum-type*
> &nbsp;&nbsp; `auto`
> &nbsp;&nbsp; `_`
> &nbsp;&nbsp; `void`
> &nbsp;&nbsp; *identifier*
>
> *fundamental-type:* one of
> &nbsp;&nbsp; `intb` `intw` `intd` `intq` `into`
> &nbsp;&nbsp; `uintb` `uintw` `uintd` `uintq` `uinto`
> &nbsp;&nbsp; `usize` `ssize`
> &nbsp;&nbsp; `floatw` `floatd` `floatq` `floato`
> &nbsp;&nbsp; `bool` `char`

---

## 6 Type Qualifiers [basic.type.qualifier]

[1] *Syntax:*

> *type-qualifier:* one of
> &nbsp;&nbsp; `mut` `imut`

[2] A type qualifier describes the mutability of a value independently of the rebindability of its binding ([dcl.var]). `mut` specifies that the value or pointed-to data may be modified. `imut` specifies that it shall not be modified after initialization. If a type qualifier is omitted, mutability is inferred from the declaration keyword of the enclosing binding according to the following table:

| Declaration keyword | Value mutability if qualifier omitted | Pointee/referent mutability if qualifier omitted |
|---|---|---|
| `const` | `imut` | `imut` |
| `let` | `imut` | `imut` |
| `var` | `mut` | `mut` |

[3] A type qualifier is part of the type, not the binding. In a pointer declaration ([basic.compound.ptr]), the qualifier on the pointed-to type and the rebindability of the pointer binding are independent properties. The qualifier on the pointer type governs whether the stored address may change; the qualifier on the pointee type governs whether the pointed-to value may be modified through the pointer.

[4] *Example:*
```
let p: ptr mut intd = addrof(x);  // p cannot be rebound; pointed-to intd is mutable
var q: ptr intd     = addrof(x);  // q can be rebound; pointed-to intd is imut (inferred)
```

---

## 7 Declarations [dcl]

### 7.1 Variable Declarations [dcl.var]

[1] A variable declaration introduces a binding between an identifier and a value.

[2] *Syntax:*

> *variable-declaration:*
> &nbsp;&nbsp; *declaration-keyword* *identifier* *visibility-modifier*_opt *generic-parameter-list*_opt *pragma*_opt *type-annotation*_opt *initializer*_opt `;`
>
> *declaration-keyword:* one of
> &nbsp;&nbsp; `const` `let` `var`
>
> *type-annotation:*
> &nbsp;&nbsp; `:` *type-qualifier*_opt *type*
>
> *initializer:*
> &nbsp;&nbsp; `=` *expression*

[3] `const` declares a binding whose value shall be evaluable at compile time. The binding shall not be rebound after declaration.

[4] `let` declares an immutable binding. The identifier shall not be rebound after the point of declaration. `let` constrains the binding; the mutability of the value is governed independently by the type qualifier ([basic.type.qualifier]).

[5] `var` declares a mutable binding. The identifier may be rebound to a new value of compatible type after the declaration.

[6] If a *type-annotation* is absent, the type of the declared variable is deduced from the initializer. A declaration with neither a *type-annotation* nor an *initializer* is ill-formed.

[7] *Example:*
```
const max: usize = 100;
let x: intd = 42;
var y = 3.14;
let xPtr: ptr mut intd = addrof(x);
let name*: [char, _] = "hello";
```

### 7.2 Type Declarations [dcl.type]

[1] A type declaration introduces a named type into the current scope.

[2] *Syntax:*

> *type-declaration:*
> &nbsp;&nbsp; `type` *identifier* *visibility-modifier*_opt *generic-parameter-list*_opt *pragma*_opt `=` *type* `;`

### 7.3 Forward Declarations [dcl.fwd]

[1] A forward declaration introduces a name into the current scope without providing a full definition. Forward declarations are required for mutual recursion and for types used before their definition in a single-pass translation unit.

[2] *Syntax:*

> *forward-declaration:*
> &nbsp;&nbsp; `fn` *identifier* *visibility-modifier*_opt `(` *parameter-list*_opt `)` *return-type*_opt `;`
> &nbsp;&nbsp; `type` *identifier* *visibility-modifier*_opt `;`
> &nbsp;&nbsp; *declaration-keyword* *identifier* *visibility-modifier*_opt `:` *type* `;`

[3] A forward-declared function shall have a matching full definition in the same translation unit or be resolved via `import` ([dcl.import]).

[4] A forward-declared type is an incomplete type. An incomplete type may be used only as the target of a pointer type. Any other use of an incomplete type is ill-formed.

[5] A forward-declared variable shall carry the visibility modifier `*` and shall be resolved via `import`. A forward-declared variable without `*` is ill-formed.

[6] A function with a deduced return type ([dcl.fct.general]) shall not be forward-declared.

[7] *Example:*
```
fn foo(x: intd): intd;  // function forward declaration
type Node;               // type forward declaration — incomplete
let x*: intd;            // variable forward declaration — resolved via import

type Node = object { val: intd, next: ptr Node, };  // full definition
```

---

## 8 The Wildcard and Anonymous Types [dcl.wild]

### 8.1 The Wildcard `_` [dcl.wild.underscore]

[1] `_` is a multi-purpose symbol whose meaning is context-dependent. Its uses are enumerated below.

[2] As a *type*, `_` instructs the compiler to generate no assembly for the declared value, regardless of the identifier used. The value is not materialized at runtime.

[3] As an *identifier* in a variable declaration, `_` makes the binding inaccessible; the value cannot be read or written after declaration, regardless of the type. If the type is not `_`, assembly is still generated for the value even though it is inaccessible.

[4] As an *array-size* expression, `_` instructs the compiler to deduce the array length from the initializer ([basic.array]).

[5] In a `for` statement ([stmt.iter]), `_` in an index or element-binding position omits that binding.

[6] In a *tuple-destructuring* ([basic.tuple]), `_` in a binding position discards the element at that position.

[7] As a scope qualifier `_::`, it refers to the immediately enclosing outer scope when a name is shadowed by a declaration in the current scope ([basic.scope.general]). The token sequence `_` `::` is always parsed as the outer-scope accessor; a standalone `_` is always the wildcard. The parser disambiguates by one token of lookahead.

[8] *Example:*
```
_ = someValue;
let _: _ = object {};
const _: void = fn();
let (a, _, c) = (1, 2, 3);
```

### 8.2 The `auto` Type [dcl.wild.auto]

[1] `auto` specifies that the declared variable has an anonymous but concrete type. A declaration of type `auto` causes the compiler to generate assembly for the object. This differs from the `_` type, for which no assembly is generated ([dcl.wild.underscore]).

[2] *Example:*
```
let _: auto = object { var x = 1, };  // ASM generated
let _: _    = object { var x = 1, };  // no ASM generated
```

---

## 9 `void` [basic.void]

[1] `void` is a first-class type representing the absence of a value. A variable of type `void` may be declared to invoke a `void`-returning function in a `const` context or to explicitly discard a result.

[2] `void` may appear as a generic argument. A parameter of type `void` accepts no value at the call site; the caller omits that argument. A function returning `void` may be used as an expression of type `void` only in discard contexts (`let _: void`, `const _: void`).

[3] *Example:*
```
fn doWork(): void {}

const _: void = doWork();
let   _: void = doWork();
```

---

## 10 Tuples [basic.tuple]

### 10.1 General [basic.tuple.general]

[1] A tuple is an ordered, fixed-length, heterogeneous sequence of values. Tuple elements are accessed by destructuring.

[2] *Syntax:*

> *tuple-type:*
> &nbsp;&nbsp; `(` *type-list* `)`
>
> *type-list:*
> &nbsp;&nbsp; *type*
> &nbsp;&nbsp; *type* `,` *type-list*
>
> *tuple-expression:*
> &nbsp;&nbsp; `(` *expression-list* `)`
>
> *tuple-destructuring:*
> &nbsp;&nbsp; *declaration-keyword* `(` *binding-list* `)` *type-annotation*_opt `=` *expression* `;`
>
> *binding-list:*
> &nbsp;&nbsp; *binding*
> &nbsp;&nbsp; *binding* `,` *binding-list*
>
> *binding:*
> &nbsp;&nbsp; *identifier*
> &nbsp;&nbsp; `_`

### 10.2 Multiple Return Values [basic.tuple.return]

[1] A function may declare a *tuple-type* as its return type to return multiple values simultaneously.

[2] *Example:*
```
fn multiReturn(): (intd, floatd, char)
{
    return (1, 2.1, 'a');
}

let (a, _, c): (intd, _, char) = multiReturn();
```

---

## 11 Arrays and Ranges [basic.array]

### 11.1 Arrays [basic.array.general]

[1] An array is a fixed-length, contiguous sequence of elements of a single type. The address of an array is immutable. The mutability of elements is governed by the declaration keyword and type qualifier of the enclosing binding. Arrays are not implicitly null-terminated.

[2] *Syntax:*

> *array-type:*
> &nbsp;&nbsp; `[` *type* `,` *array-size* `]`
>
> *array-size:*
> &nbsp;&nbsp; *expression*
> &nbsp;&nbsp; `_`

[3] When the *array-size* is `_`, the array length is deduced from the initializer. It is ill-formed to declare an array with size `_` and no initializer.

[4] A string literal ([lex.literal]) used as an initializer for a `char` array initializes the array with the character sequence of the literal. The array does not receive a null terminator unless `\0` appears explicitly in the literal.

[5] *Example:*
```
let name: [char, _] = "hello";
let nums: [intd, _] = [1, 2, 3];
```

### 11.2 Ranges [basic.array.range]

[1] A range expression denotes a contiguous sequence of values between two endpoint expressions. A range is not a type and shall not appear as a general expression. A range expression is valid only inside `[` `]` to produce an array literal, or as a *case-label-expr* in a `case` statement ([stmt.case]).

[2] *Syntax:*

> *range-expression:*
> &nbsp;&nbsp; *expression* `..` *expression*

[3] The expression `[` *N1* `..` *N2* `]` where *N1* ≤ *N2* produces an ascending array containing all values from *N1* to *N2* inclusive.

[4] If *N1* is greater than *N2*, the range produces a descending array from *N1* to *N2* inclusive.

[5] The `..` token has no operator precedence; it is a grammatical production only. Its use outside the contexts described in [11.2.1] is ill-formed.

[6] *Example:*
```
let arr = [0 .. 5];  // produces [0, 1, 2, 3, 4, 5]
```

---

## 12 Pointers, References, and Values [basic.compound]

### 12.1 Pointers [basic.compound.ptr]

[1] A pointer holds a raw memory address of an object of its pointed-to type.

[2] *Syntax:*

> *pointer-type:*
> &nbsp;&nbsp; `ptr` *type-qualifier*_opt *type*

[3] There is no unary dereference operator. The pointed-to value shall be accessed via array subscript syntax: `p[0]` denotes the value at the address held by `p`.

[4] When a pointer to `char` is passed to a function expecting a null-terminated string, the user is responsible for ensuring null termination. The language does not impose null termination.

[5] *Note: The language defines no null pointer literal. A null pointer may be constructed as `cast<ptr void>(usize(0))`. Convenience aliases are expected to be provided by the standard library.*

[6] *Example:*
```
var x: intd = 1;
var xPtr: ptr mut intd = addrof(x);
xPtr[0] = 2;
```

### 12.2 The `addrof` Function [basic.compound.addr]

[1] `addrof` is a built-in function that yields the address of its operand as a pointer of the appropriate type. `addrof` shall not be overloaded. See ([over.reflect]) for the full list of built-in functions.

[2] The operand of `addrof` shall be an lvalue.

[3] A `const` binding shall not be used as the operand of `addrof`. `const` bindings have no runtime address.

### 12.3 References [basic.compound.ref]

[1] A reference is an alias for an existing object. Assignment through a reference modifies the referenced object directly.

[2] *Syntax:*

> *reference-type:*
> &nbsp;&nbsp; `ref` *type-qualifier*_opt *type*

[3] A reference shall be initialized at its point of declaration and shall not be rebound thereafter, regardless of the declaration keyword.

[4] *Example:*
```
var x: intd = 1;
var xRef: ref intd = x;
xRef = 2;  // modifies x
```

### 12.4 Values [basic.compound.val]

[1] A value type binds to an rvalue; it holds a temporary or moved value.

[2] *Syntax:*

> *value-type:*
> &nbsp;&nbsp; `val` *type-qualifier*_opt *type*

[3] *Example:*
```
let r: val intw = 1;
```

---

## 13 Objects [class]

### 13.1 General [class.general]

[1] An object type is a record of named fields. Fields are private to the translation unit by default. The visibility modifier `*` exports a field ([lex.vis]). Fields are parameter bindings ([intro.defs]).

[2] *Syntax:*

> *object-type:*
> &nbsp;&nbsp; `object` *inheritance-list*_opt `{` *field-seq*_opt `}`
>
> *inheritance-list:*
> &nbsp;&nbsp; `,` *identifier*
> &nbsp;&nbsp; `,` *identifier* *inheritance-list*
>
> *field-seq:*
> &nbsp;&nbsp; *field*
> &nbsp;&nbsp; *field* *field-seq*
>
> *field:*
> &nbsp;&nbsp; *declaration-keyword*_opt *identifier* *visibility-modifier*_opt *pragma*_opt *type-annotation*_opt *initializer*_opt `,`

### 13.2 Construction [class.ctor]

[1] An object is constructed with named arguments only. Positional construction is ill-formed. The order of named arguments is not significant.

[2] *Example:*
```
type Data = object { value*: intd, };
let d = Data(value: 42);
```

### 13.3 Inheritance [class.inherit]

[1] An object type may inherit from one or more named object types by listing them in the *inheritance-list*.

### 13.4 Immediate Objects [class.immediate]

[1] An anonymous object may be declared inline without a prior *type-declaration* ([dcl.type]).

[2] *Example:*
```
let obj = object { let x* = 1, var y = 2, };
```

### 13.5 `this` and `This` [class.this]

[1] Within an object body, `this` is an implicitly available identifier of type `ptr` to the enclosing object type. `This` is an implicitly available type alias for the enclosing object type.

[2] `this` and `This` shall not be used outside of an object body. Their use in free functions, including UFCS functions ([over.call]), is ill-formed.

[3] *Example:*
```
type Counter = object {
    var count*: intd = 0,
    let increment = fn() { this->count += 1; },
    let self: This,
};
```

### 13.6 Field Initializers [class.init]

[1] Field initializers are evaluated per construction, in declaration order, each time an object instance is created.

[2] A field declared `const` shall have a constant expression initializer. `const` field initializers are evaluated once at type-definition time and are shared across all instances.

[3] *Example:*
```
var counter: intd = 0;

type Foo = object {
    var x = (counter += 1),  // evaluated each construction
    const id = 99,            // evaluated once at type definition
};

let a = Foo();  // counter becomes 1; a.x = 1
let b = Foo();  // counter becomes 2; b.x = 2
// a.id == b.id == 99
```

---

## 14 Enumerations [dcl.enum]

### 14.1 General [dcl.enum.general]

[1] An enumeration is a discriminated type whose variants each carry either a fixed value or a type. Every variant shall be assigned a value or a type; a variant with neither is ill-formed.

[2] *Syntax:*

> *enum-type:*
> &nbsp;&nbsp; `enum` `{` *variant-seq* `}`
>
> *variant-seq:*
> &nbsp;&nbsp; *variant*
> &nbsp;&nbsp; *variant* *variant-seq*
>
> *variant:*
> &nbsp;&nbsp; *identifier* `=` *expression* `,`
> &nbsp;&nbsp; *identifier* `=` *type* `,`

[3] Variants are accessed via `::` ([lex.scope]).

[4] Each variant of an enumeration carries an implicit unique integer discriminant assigned by the compiler in declaration order, beginning at zero. The discriminant is used by `case` ([stmt.case]) to identify the active variant at runtime.

### 14.2 Value Variants [dcl.enum.value]

[1] A value variant assigns a fixed expression to a variant name. Accessing the variant via `::` yields the assigned value directly.

[2] *Example:*
```
type Person = enum {
    Student = fn(): [char, _] { return "I am a student!"; },
    Worker  = fn(): [char, _] { return "I am a worker!"; },
};

let student = Person::Student;   // yields the lambda
let result  = Person::Worker();  // calls the lambda; holds "I am a worker!"
```

### 14.3 Type Variants [dcl.enum.type]

[1] A type variant associates a variant name with a type. A value of that type may be wrapped into the variant using variant construction syntax. The wrapped value may be extracted through pattern matching ([stmt.case]).

[2] *Syntax:*

> *variant-construction-expression:*
> &nbsp;&nbsp; *qualified-variant* `(` *expression* `)`
>
> *qualified-variant:*
> &nbsp;&nbsp; *identifier* `::` *identifier*

[3] A *variant-construction-expression* produces a value of the enclosing enumeration type with the specified variant active and the given expression as the inner value.

[4] *Example:*
```
type Number = enum {
    Int   = intd,
    Float = floatd,
};

let n = Number::Int(42);      // wraps 42 as the Int variant
let f = Number::Float(3.14);  // wraps 3.14 as the Float variant
```

---

## 15 Functions and Generics [dcl.fct]

### 15.1 Function Declarations [dcl.fct.general]

[1] *Syntax:*

> *function-declaration:*
> &nbsp;&nbsp; *eval-prefix*_opt `fn` *identifier* *visibility-modifier*_opt *generic-parameter-list*_opt `(` *parameter-list*_opt `)` *pragma*_opt *return-type*_opt *compound-statement*
>
> *eval-prefix:* one of
> &nbsp;&nbsp; `const` `template`
>
> *generic-parameter-list:*
> &nbsp;&nbsp; `<` *generic-parameter-seq* `>`
>
> *generic-parameter-seq:*
> &nbsp;&nbsp; *generic-parameter*
> &nbsp;&nbsp; *generic-parameter* `,` *generic-parameter-seq*
>
> *generic-parameter:*
> &nbsp;&nbsp; *identifier*
> &nbsp;&nbsp; *identifier* `:` *generic-type*
> &nbsp;&nbsp; *identifier* `:` *generic-type* `=` *type*
>
> *generic-type:*
> &nbsp;&nbsp; `type`
> &nbsp;&nbsp; *array-type*
>
> *parameter-list:*
> &nbsp;&nbsp; *parameter*
> &nbsp;&nbsp; *parameter* `,` *parameter-list*
>
> *parameter:*
> &nbsp;&nbsp; *declaration-keyword*_opt *identifier* *type-annotation*_opt *initializer*_opt
>
> *return-type:*
> &nbsp;&nbsp; `:` *type*
>
> *return-statement:*
> &nbsp;&nbsp; `return` *expression*_opt `;`

[2] If a *return-type* is absent, the return type is deduced from the operand of the `return` statement. All `return` statements in the function shall yield expressions of the same type. A function whose body contains no `return` statement with an operand has deduced return type `void`. A function with a deduced return type shall not be forward-declared ([dcl.fwd]).

[3] Function parameters are parameter bindings ([intro.defs]). If a parameter has no *type-annotation*, its type is deduced from the *initializer*. A parameter with neither a *type-annotation* nor an *initializer* is ill-formed.

[4] A *generic-parameter* is an implicitly `const` binding whose `const` declarator is not written and cannot be specified. The *generic-type* annotation defaults to `type` if omitted. Generic parameters are the only context in which a variable binding may hold a value of type `type`; `type` is not available as a type annotation outside of *generic-parameter* and `type` declarations ([dcl.type]).

### 15.2 Lambdas [dcl.fct.lambda]

[1] A lambda is an anonymous function expression. Its type is `auto`.

[2] *Syntax:*

> *lambda-expression:*
> &nbsp;&nbsp; `fn` *generic-parameter-list*_opt `(` *parameter-list*_opt `)` *return-type*_opt *compound-statement*

[3] *Example:*
```
let add = fn(x: intd, y: intd): intd { return x + y; };
let result = add(1, 2);
```

### 15.3 Type-as-Constructor [dcl.fct.typecast]

[1] A fundamental or user-defined type may be used as a constructor to produce a value of that type from an expression. This performs a non-reinterpret conversion and is distinct from `cast` ([expr.cast]).

[2] *Syntax:*

> *type-constructor-expression:*
> &nbsp;&nbsp; *type* `(` *expression* `)`

[3] *Example:*
```
let x = floatd(1);    // converts intd 1 to floatd
let y = intd(3.14);   // truncates to intd
```

### 15.4 Generics [dcl.fct.generic]

[1] Generic parameters are implicitly `const` and may appear anywhere a type is permitted in the function signature or body. A generic parameter's *generic-type* annotation is optional and defaults to `type` ([basic.types.type]).

[2] A generic function is instantiated once per unique combination of generic arguments at the point of first call. Instantiation consists of substituting each generic parameter with the supplied argument, followed by full semantic analysis of the resulting body. If semantic analysis of an instantiation fails, the program is ill-formed at the call site. Two or more calls with identical generic arguments share one instantiation.

[3] A generic parameter with a default type shall not precede a generic parameter without a default type. A variadic generic parameter shall be the last in the *generic-parameter-list* and is declared by annotating the parameter with an array type of deduced size. Variadic arguments may be iterated as an array.

[4] *Example:*
```
fn swap<T>(a: ref T, b: ref T): void
{
    let tmp: T = a;
    a = b;
    b = tmp;
}

fn sum<T: [type, _]>(args: T): intd
{
    var total: intd = 0;
    for v, args { total += intd(v); }
    return total;
}

fn defaulted<T: type = intd>(x: T): T { return x; }
```

---

## 16 Templates [dcl.template]

### 16.1 General [dcl.template.general]

[1] The `template` keyword applied to a construct instructs the compiler to expand it textually at the point of use. Templates are implicitly evaluated at compile time. Combining `template` with `const` on the same declaration is ill-formed.

### 16.2 Template Functions [dcl.template.fn]

[1] A template function is declared with `template fn`. Its body is textually embedded at every call site. A template function shall not declare a *return-type*.

[2] A parameter of type `[char, _]` in a template function may receive an argument in two ways, determined by the parameter's type:

- If the parameter is declared as `[char, _]` (by value), the compiler first attempts to resolve the argument as a variable in the current scope. If a variable of type `[char, N]` is found, its value is passed. If no such variable is found, the identifier is stringified and passed as a `[char, N]` literal. If a variable is found but is not of type `[char, N]`, the program is ill-formed.
- If the parameter is declared as `ref [char, _]`, the argument shall be an lvalue of type `[char, N]`. Stringification does not occur; no variable of that name is required to be of char array type.

[3] If the last parameter is of type `[char, _]`, the caller may omit the parenthesized argument for that parameter and supply it as a `{}`-delimited block following the call. The entire contents of the block are passed as a raw, unevaluated string.

[4] *Example:*
```
template fn myClass(symbol: [char, _], body: [char, _])
{
    mixin("type " + symbol + " = object {" + body + "};");
}

myClass Person
{
    var age*: intd = 0,
};

// passing a char array variable explicitly
let typeName: [char, _] = "Animal";
template fn declare(name: ref [char, _]) { mixin("type " + name + " = object {};"); }
declare typeName;  // passes the variable; stringification does not occur
```

### 16.3 Template Loops [dcl.template.loop]

[1] `template for` and `template while` unroll their bodies at compile time, embedding each iteration textually at the call site. Their syntax is identical to `for` and `while` ([stmt.iter]) with `template` prepended.

[2] *Example:*
```
template fn repeat(times: usize)
{
    template for _, [1 .. times] { }
}
```

---

## 17 Mixins [dcl.mixin]

[1] `mixin` is a built-in function that converts a string expression to a sequence of GAL tokens and inserts those tokens at the point of the call.

[2] The operand shall evaluate to a value of type `[char, N]` for some `N`. The resulting token sequence shall be syntactically valid in the context of the insertion point.

[3] *Example:*
```
mixin("let x = 1;");

const src = "let y = 2;";
mixin(src);
```

---

## 18 Control Flow [stmt]

### 18.1 General [stmt.general]

[1] *Syntax:*

> *statement:*
> &nbsp;&nbsp; *expression-statement*
> &nbsp;&nbsp; *compound-statement*
> &nbsp;&nbsp; *variable-declaration*
> &nbsp;&nbsp; *if-statement*
> &nbsp;&nbsp; *while-statement*
> &nbsp;&nbsp; *for-statement*
> &nbsp;&nbsp; *case-statement*
> &nbsp;&nbsp; *label-declaration*
> &nbsp;&nbsp; *goto-statement*
> &nbsp;&nbsp; *return-statement*
> &nbsp;&nbsp; *break-statement*
> &nbsp;&nbsp; *continue-statement*
>
> *compound-statement:*
> &nbsp;&nbsp; `{` *statement-seq*_opt `}`
>
> *statement-seq:*
> &nbsp;&nbsp; *statement*
> &nbsp;&nbsp; *statement* *statement-seq*
>
> *break-statement:*
> &nbsp;&nbsp; `break` `;`
>
> *continue-statement:*
> &nbsp;&nbsp; `continue` `;`

[2] Every statement shall be terminated by `;` except *compound-statement*, *if-statement*, *while-statement*, *for-statement*, and *case-statement*.

### 18.2 Selection Statements [stmt.if]

[1] *Syntax:*

> *if-statement:*
> &nbsp;&nbsp; *if-prefix*_opt `if` `(` *expression* `)` *compound-statement*
> &nbsp;&nbsp; *if-prefix*_opt `if` `(` *expression* `)` *compound-statement* *elif-chain*
> &nbsp;&nbsp; *if-prefix*_opt `if` `(` *expression* `)` *compound-statement* *elif-chain*_opt `else` *compound-statement*
> &nbsp;&nbsp; `if` `const` *compound-statement*
>
> *if-prefix:* one of
> &nbsp;&nbsp; `const` `template`
>
> *elif-chain:*
> &nbsp;&nbsp; `elif` `(` *expression* `)` *compound-statement*
> &nbsp;&nbsp; `elif` `(` *expression* `)` *compound-statement* *elif-chain*

[2] `const if` evaluates the condition at compile time. The branch not taken is not instantiated.

[3] `if const` executes the body only when the current evaluation context is compile time ([intro.eval]).

[4] `template if` embeds the body of the selected branch textually into the enclosing scope.

[5] When an `if` statement is prefixed with `const` or `template`, all associated `elif` and `else` branches are implicitly evaluated under the same prefix.

### 18.3 Iteration Statements [stmt.iter]

[1] *Syntax:*

> *while-statement:*
> &nbsp;&nbsp; *iter-prefix*_opt `while` *expression* *compound-statement*
> &nbsp;&nbsp; *iter-prefix*_opt `while` *for-init* `,` *expression* `,` *expression* *compound-statement*
>
> *for-statement:*
> &nbsp;&nbsp; *iter-prefix*_opt `for` *for-index*_opt *element-binding* `,` *expression* *compound-statement*
>
> *iter-prefix:* one of
> &nbsp;&nbsp; `const` `template`
>
> *for-init:*
> &nbsp;&nbsp; *declaration-keyword*_opt *identifier* *type-annotation*_opt `=` *expression*
>
> *for-index:*
> &nbsp;&nbsp; *declaration-keyword*_opt *identifier* *type-annotation*_opt `=` *expression* `,`
> &nbsp;&nbsp; `_` `,`
>
> *element-binding:*
> &nbsp;&nbsp; *declaration-keyword*_opt *identifier* *type-annotation*_opt
> &nbsp;&nbsp; `_`

[2] In a three-part `while`, the first operand is the initializer, the second is the loop condition, and the third is the update expression, each separated by `,`.

[3] The simple while repeatedly evaluates its condition and executes the body as long as the condition is non-zero. The three-part while executes the initializer once, then repeatedly checks the condition and executes the body followed by the update expression.

[4] In a `for` statement, the *element-binding* and the iterated *expression* are mandatory. The *for-index* may be omitted. `_` in any binding position discards that binding. If both for-index and element-binding are `_`, the loop iterates over the expression without binding either. For-index and element-binding are parameter bindings ([intro.defs]).

[5] The for statement evaluates the iterated expression once to obtain a sequence. If the expression returns a `(T, usize)` tuple via a for overload ([over.for]), the body executes `usize` times, binding each element in turn. If `T` is `ptr U`, the element advances by `sizeof<U>()` bytes per iteration; otherwise by `sizeof<T>()`. If the iterated expression is an array, the body executes `lenof(array)` times.

[6] The following declarations are equivalent:
```
for e, [1, 2, 3] {}
for let e, [1, 2, 3] {}
for let e: intd, [1, 2, 3] {}
```

[7] `break` exits the innermost enclosing loop. `continue` advances to the next iteration. Fall-through between iterations does not occur implicitly.

---

## 19 Labels and Goto [stmt.label]

[1] A label marks a location in the instruction stream. A `goto` statement transfers control unconditionally to a label or a raw address.

[2] *Syntax:*

> *label-declaration:*
> &nbsp;&nbsp; `label` *identifier* `;`
>
> *goto-statement:*
> &nbsp;&nbsp; `goto` *identifier* `;`
> &nbsp;&nbsp; `goto` *integer-literal* `;`

[3] No two labels in the same translation unit shall share an identifier.

[4] A raw address may be stored in an appropriately sized integral variable and used as the operand of `goto`. The behavior of `goto` applied to an address that does not denote a valid instruction boundary is undefined ([intro.undef]).

[5] *Example:*
```
label myLabel;
goto myLabel;
goto 0xDEADBEEF;
```

---

## 20 Pattern Matching [stmt.case]

[1] A `case` statement is a scoped computed-goto construct. It evaluates its operand and transfers control to the matching label within its body.

[2] *Syntax:*

> *case-statement:*
> &nbsp;&nbsp; `case` `(` *expression* `)` `{` *case-body* `}`
>
> *case-body:*
> &nbsp;&nbsp; *case-label-seq*_opt
>
> *case-label-seq:*
> &nbsp;&nbsp; *case-label*
> &nbsp;&nbsp; *case-label* *case-label-seq*
>
> *case-label:*
> &nbsp;&nbsp; `label` *case-label-expr* `:` *statement-seq*_opt
>
> *case-label-expr:*
> &nbsp;&nbsp; *expression*
> &nbsp;&nbsp; *range-expression*
> &nbsp;&nbsp; *variant-pattern*
> &nbsp;&nbsp; `_`
>
> *variant-pattern:*
> &nbsp;&nbsp; *qualified-variant* `(` *identifier* `)`
> &nbsp;&nbsp; *qualified-variant*

[3] `_` as a *case-label-expr* matches any value not matched by a preceding label. It is the default arm.

[4] A *range-expression* as a *case-label-expr* matches any value within the specified range inclusive.

[5] A *variant-pattern* matches when the active variant of the `case` operand equals the specified variant. If the variant pattern includes an *identifier* in parentheses, that identifier is bound to the inner value of the matched type variant for the duration of the arm's *statement-seq*. The bound identifier follows `let` binding semantics.

[6] Fall-through does not occur. Control exits the `case` statement at the end of each arm's *statement-seq* unless transferred by other means. `break` may be used to exit the `case` body early or to exit an enclosing loop from within a `case` arm.

[7] *Note: The `case` statement is built on the label and goto mechanisms ([stmt.label]). Labels within a `case` body are local to that body.*

[8] A case body shall contain at most one `label _:` arm. A case body with no labels is valid and transfers control past the case statement.

[9] *Example — integer and range matching:*
```
case(x)
{
    label _:
        print("not found");

    label 0 .. 10:
        print("in range");

    label 42:
        print("exactly 42");
}
```

[10] *Example — enum value variant matching:*
```
type Person = enum {
    Student = fn(): [char, _] { return "I am a student!"; },
    Worker  = fn(): [char, _] { return "I am a worker!"; },
};

let p = Person::Student;

case(p)
{
    label Person::Student:
        print("student");
    label Person::Worker:
        print("worker");
}
```

[11] *Example — enum type variant matching with binding:*
```
type Number = enum {
    Int   = intd,
    Float = floatd,
};

let n = Number::Int(42);

case(n)
{
    label Number::Int(v):
        print(v + 1);
    label Number::Float(v):
        print(v * 2.0);
}
```

---

## 21 Specialized Symbols and Operator Overloading [over]

### 21.1 General [over.general]

[1] Specialized symbols are operators and built-in constructs that use a calling convention specific to their syntactic position rather than standard function call syntax (e.g., `a + b`, `for i, e, c {}`).

[2] Most specialized symbols may be overloaded by defining a function whose name is the symbol enclosed in backticks. Built-in functions ([over.reflect]) shall not be overloaded.

[3] *Syntax:*

> *operator-overload:*
> &nbsp;&nbsp; `fn` `` ` `` *operator-symbol* `` ` `` *generic-parameter-list*_opt `(` *parameter-list* `)` *return-type*_opt *compound-statement*

[4] *Example:*
```
fn `+`<T>(left: T, right: T): T { ... }
fn `-`<T>(left: T, right: T): T { ... }
```

### 21.2 `for` Overloading [over.for]

[1] A type may be made iterable by overloading the `for` symbol. The overload shall accept the iterable object as its sole parameter and shall return a two-element tuple of exactly type `(T, usize)`, where `T` is the element type and `usize` is the element count. If `T` is a pointer type, elements are accessed via subscript; otherwise the value is used directly.

[2] *Example:*
```
type Data = object {
    data: ptr uintb,
    length: usize,
};

fn `for`(this: Data): (ptr uintb, usize)
{
    return (this.data, this.length);
}
```

---

## 22 Scopes and Aliases [basic.scope]

### 22.1 Scopes [basic.scope.general]

[1] A compound statement ([stmt.general]) introduces a new scope. Names declared within a scope are local to it unless marked with the visibility modifier `*`, which exports them to the immediately enclosing scope.

[2] When a name declared inside a scope shadows a name in an enclosing scope, the outer name remains accessible as `_::` *identifier* ([dcl.wild.underscore]).

[3] *Example:*
```
let x = 1;
{
    let x = 2;
    let y* = x + _::x;  // y = 3; exported
}
print(y);  // valid
```

### 22.2 Aliases [basic.scope.alias]

[1] An `alias` declaration introduces a new name for an existing declaration, scope, or sub-member.

[2] *Syntax:*

> *alias-declaration:*
> &nbsp;&nbsp; `alias` *identifier* *visibility-modifier*_opt `=` *alias-target* `;`
>
> *alias-target:*
> &nbsp;&nbsp; *expression*
> &nbsp;&nbsp; *scope-expression*
>
> *scope-expression:*
> &nbsp;&nbsp; `{` *statement-seq*_opt `}`

[3] When `_` is the alias identifier, the alias has no name and all exported members of the aliased scope are injected directly into the enclosing scope.

[4] It is ill-formed for two unnamed aliases in the same enclosing scope to export members with conflicting names.

[5] *Example:*
```
alias myScope* = { var x* = 1; };
myScope::x = 2;

alias _ = myScope;
x = 3;
```

---

## 23 Import and Include [dcl.import]

### 23.1 `include` [dcl.import.include]

[1] An `include` directive is replaced textually by the contents of the named source file at the point of inclusion. All declarations in the included file become available in the current translation unit regardless of their visibility.

[2] *Syntax:*

> *include-directive:*
> &nbsp;&nbsp; `include` *string-literal* `;`

### 23.2 `import` [dcl.import.import]

[1] An `import` directive makes all declarations in the named translation unit available in the current translation unit. Declarations without `*` are local to the importing translation unit and shall not be re-exported. Declarations with `*` are re-exported only when the import itself carries `*` ([lex.vis]).

[2] *Syntax:*

> *import-directive:*
> &nbsp;&nbsp; `import` *string-literal* *visibility-modifier*_opt `;`

[3] When `*` follows the string literal, all imported names are re-exported to any translation unit that subsequently imports the current file.

[4] *Example:*
```
// A.gal
let data1* = object { d1*: intb, d2: intb, };
let data2  = object {};

// main.gal
import "A.gal";    // data1 and data1.d1 available; data2 and data1.d2 are not

// main2.gal
include "A.gal";   // all declarations available
```

---

## 24 Casts [expr.cast]

[1] A cast reinterprets the bit representation of its operand as a different type. No arithmetic conversion is performed.

[2] *Syntax:*

> *cast-expression:*
> &nbsp;&nbsp; `cast` `<` *type* `>` `(` *expression* `)`

[3] If the size of the source type and the size of the target type differ, the behavior is undefined ([intro.undef]).

[4] *Example:*
```
let x: intd = 65;
let c: char = cast<char>(x);
```

---

## 25 Function Call Syntax [over.call]

[1] Any function whose first parameter is of type *T* may be called on a value of type *T* using member-access syntax. This is called Uniform Function Call Syntax (UFCS).

[2] *Syntax:*

> **call:**
>
> - *identifier* *generic-parameter-list*ₒₚₜ `(` *argument-list*ₒₚₜ `)` `;`
> - *identifier* *generic-parameter-list*ₒₚₜ *argument-list* `;`
>
> - *expression* `.` *identifier* *generic-parameter-list*ₒₚₜ `(` *argument-list*ₒₚₜ `)` `;`
> - *expression* `.` *identifier* *generic-parameter-list*ₒₚₜ *argument-list* `;`
>
> **argument-list:**
>
> - *expression*
> - *expression* `,` *argument-list*

[3] `x.f(y)` is equivalent to `f(x, y)`. The value to the left of `.` is passed as the first argument.

[4] UFCS shall not apply to specialized symbols ([over.general]) or built-in functions ([over.reflect]). Operator overloads, `for`, and all other specialized symbols may only be invoked through their prescribed syntactic form.

[5] If both a field of an object and a UFCS-eligible free function share the same name for a given type, and the call is syntactically ambiguous, the program is ill-formed.

[6] The parenthesized form `expression.identifier()` is required when there are no additional arguments beyond the receiver because `expression.identifier` without `(` is grammatically a field access expression and is resolved as such by the parser. The presence of `(` disambiguates a UFCS call from a field access.

[7] *Example:*
```
fn sum(x: intd, y: intd, z: intd): intd { return x + y + z; }

let reg         = sum(1, 2, 3);
let split       = sum 1, 2, 3;
let method      = 1.sum(2, 3);
let splitmethod = 1.sum 2, 3;
```

---

## 26 Operator Precedence [expr.prec]

[1] The following table defines operator precedence from highest (1) to lowest. Operators on the same row share the same precedence. Associativity is left-to-right unless marked R (right-to-left).

| Level | Operators | Associativity |
|-------|-----------|---------------|
| 1 | `()` `[]` `.` `->` `::` | L |
| 2 | Unary `!` `~` `-` `+` `cast` | R |
| 3 | `*` `/` `%` | L |
| 4 | `+` `-` | L |
| 5 | `<<` `>>` | L |
| 6 | `<` `>` `<=` `>=` | L |
| 7 | `==` `!=` | L |
| 8 | `&` | L |
| 9 | `^` | L |
| 10 | `&&` | L |
| 11 | `\|\|` | L |
| 12 | `=` `+=` `-=` `*=` `/=` `%=` `&=` `\|=` `^=` `<<=` `>>=` | R |

[2] The `|` token is context-sensitive. In an *expression* context it is bitwise OR and participates in the precedence table above between levels 9 and 10:

| Level | Operators | Associativity |
|-------|-----------|---------------|
| 9 | `^` | L |
| 9.5 | `\|` | L |
| 10 | `&&` | L |

[3] The `..` token is not an operator and has no precedence. It is a grammatical production valid only inside `[` `]` array literals and `case` label positions ([basic.array.range], [stmt.case]).

---

## 27 Pragmas [dcl.pragma]

### 27.1 General [dcl.pragma.general]

[1] A pragma is a compile-time directive that provides additional information to the compiler about a declaration or statement. Pragmas do not alter the semantics of a program but may affect code generation, diagnostics, or linkage.

[2] *Syntax:*

> *pragma:*
> &nbsp;&nbsp; `[.` *pragma-list* `.]`
>
> *pragma-list:*
> &nbsp;&nbsp; *pragma-item*
> &nbsp;&nbsp; *pragma-item* `,` *pragma-list*
>
> *pragma-item:*
> &nbsp;&nbsp; *pragma-name*
> &nbsp;&nbsp; *pragma-name* `(` *expression* `)`
> &nbsp;&nbsp; *pragma-name* `:` *specifier*
> &nbsp;&nbsp; *pragma-name* `:` *specifier* `(` *expression* `)`

[3] A pragma shall appear immediately after the identifier it applies to, with the following permitted tokens between the identifier and the pragma:

- For all declarations: the visibility modifier `*` may appear between the identifier and the pragma.
- For function declarations: the parameter list shall appear between the identifier and the pragma, optionally preceded by `*`.

A pragma shall not appear before a declaration. The following forms are the only valid pragma placements:
```
var x [.volatile.]: intd = 0;
var x* [.volatile.]: intd = 0;
type Flags [.packed.] = object { ... };
type Flags* [.packed.] = object { ... };
fn retInt() [.inline: always.]: intd { ... }
type Flags = object {
    active* [.bit: set(1).]: uintb = 0,
};
```

### 27.2 Standalone Pragmas [dcl.pragma.standalone]

[1] The following pragmas take no arguments:

- `[.deprecated.]` — marks a declaration as deprecated; the compiler shall emit a diagnostic at each use site.
- `[.packed.]` — suppresses padding between fields of an object.
- `[.noreturn.]` — declares that the function never returns.
- `[.static.]` — declares a field or function as belonging to the type rather than any instance.
- `[.volatile.]` — prevents the compiler from caching or eliminating reads and writes to the declared variable.
- `[.cold.]` — hints that the function or branch is rarely executed.
- `[.hot.]` — hints that the function or branch is frequently executed.

### 27.3 Expression Pragmas [dcl.pragma.expr]

[1] The following pragmas take a single expression argument:

- `[.align(N).]` — sets the memory alignment of the declaration to *N* bytes.
- `[.deprecated("msg").]` — deprecation with a diagnostic message.
- `[.section("name").]` — places the declaration in the named memory section.
- `[.warning("msg").]` — emits a compile-time warning at the point of use.
- `[.error("msg").]` — renders the program ill-formed with the given message at the point of use.
- `[.export("name").]` — exports the symbol under the given name in the output binary.
- `[.import("name").]` — imports the symbol by the given name from an external binary.
- `[.convention("name").]` — sets the calling convention of the function.

### 27.4 Specifier Pragmas [dcl.pragma.specifier]

[1] The following pragmas take a specifier:

- `[.inline: default.]` — lets the compiler decide whether to inline the function.
- `[.inline: always.]` — forces the function to be inlined at every call site.
- `[.inline: never.]` — prevents the function from being inlined.
- `[.header: guard.]` — equivalent to `#pragma once`; prevents multiple inclusion of a file.
- `[.header: public.]` — all declarations following this pragma in the file are exported.
- `[.header: private.]` — all declarations following this pragma in the file are private.
- `[.asmStackframe: on.]` — enables compiler-generated stack frame for the function.
- `[.asmStackframe: off.]` — disables compiler-generated stack frame.

### 27.5 Specifier-Expression Pragmas [dcl.pragma.specexpr]

[1] The following pragmas take both a specifier and an expression:

- `[.bit: get(` *type* `).]` — initializes the declared variable with the bit size of the given type. The declared variable shall be of type `usize` and shall not have an initializer. Evaluated at compile time.
- `[.bit: set(` *N* `).]` — constrains a field to occupy exactly *N* bits within its containing object.

[2] *Example:*
```
fn retInt() [.inline: always.]: intd { return 42; }

type Data = object
{
    data* [.bit: set(6).]: uintb = 0,
};

let bitSize [.bit: get(intd).]: usize;
```

---

## 28 Inline Assembly [stmt.asm]

[1] An `asm` statement embeds assembly instructions directly into the output at the point of use. `asm` is a specialized symbol ([over.general]) and shall not be overloaded.

[2] *Syntax:*

> *asm-statement:*
> &nbsp;&nbsp; `asm` `(` *asm-string* `,` *asm-operand-list* `,` *asm-operand-list* `,` *asm-operand-list* `)` `;`
>
> *asm-string:*
> &nbsp;&nbsp; *expression*
>
> *asm-operand-list:*
> &nbsp;&nbsp; `_`
> &nbsp;&nbsp; *asm-operand*
> &nbsp;&nbsp; *asm-operand* `,` *asm-operand-list*
>
> *asm-operand:*
> &nbsp;&nbsp; *expression*

[3] The *asm-string* shall evaluate to a value of type `[char, N]`. The operand format follows the GAS extended inline assembly convention. `_` in any operand position indicates that operand is absent.

[4] The assembly dialect accepted is implementation-defined ([intro.impldef]).

[5] *Example:*
```
asm("mov rax, 1", _, _, _);

const instr = "ret";
asm(instr, _, _, _);
```

---

## 29 Built-in Functions [over.reflect]

### 29.1 General [over.reflect.general]

[1] GAL provides a set of built-in functions for compile-time reflection, type querying, and code generation. Built-in functions shall not be overloaded and UFCS shall not apply to them ([over.call]).

### 29.2 Identity and Stringification [over.reflect.identity]

[1] The following built-in functions operate on identifiers and expressions as text:

- `stringof(` *expr* `)` — yields *expr* as a `[char, N]` array without evaluating it.
- `identof(` *expr* `)` — yields the identifier name of *expr* as a `[char, N]` array.

### 29.3 Type Query Functions [over.reflect.type]

[1] The following built-in functions query properties of types:

- `typeof(` *expr* `)` — yields the type of *expr*.
- `sizeof<` *T* `>()` — yields the size of type *T* in bytes as a `usize`.
- `sizeof(` *expr* `)` — yields the size of the object *expr* in bytes as a `usize`.
- `alignof<` *T* `>()` — yields the alignment requirement of type *T* in bytes as a `usize`.
- `alignof(` *expr* `)` — yields the alignment requirement of object *expr* in bytes as a `usize`.
- `offsetof<` *T* `>(` *field* `)` — yields the byte offset of *field* within object type *T* as a `usize`.
- `hasfield<` *T* `>(` *name* `)` — yields true if type T has a field whose identifier matches name, false otherwise. name shall be a constant expression of type [char, N]. hasfield considers all fields of T regardless of their visibility modifier. Inherited fields are included. Evaluated at compile time.

### 29.4 Length and Field Count [over.reflect.len]

[1] The following built-in functions query the element or field count of a type or value:

- `lenof(` *expr* `)` — yields the element count of an array or the total field count of an object or enum (including inherited fields) as a `usize`.
- `lenof<` *T* `>()` — yields the total field count of object or enum type *T* (including inherited fields) as a `usize`.

[2] Fields are counted in base-first declaration order: fields inherited from base types are counted before the fields declared directly in *T*. Within each type in the chain, fields are counted in declaration order.

### 29.5 Object Reflection [over.reflect.object]

[1] The following built-in functions reflect over object types:

- `fieldof<` *T* `>(` *index* `)` — yields the field of type *T* at the given index as a variable; suitable for iteration and direct manipulation. Fields are ordered base-first as described in ([over.reflect.len]). The *index* argument shall be a constant expression. The return type is the statically known declared type of the field at position *index* in *T*. If *index* is out of range, the program is ill-formed.

[2] `fieldof` is primarily useful in `template for` loops where the index is a compile-time constant at each unrolled iteration.

### 29.6 Address [over.reflect.addr]

[1] The following built-in function yields the address of its operand:

- `addrof(` *expr* `)` — yields the address of *expr* as a pointer of the appropriate type. The operand shall be an lvalue. A `const` binding shall not be used as the operand. See ([basic.compound.addr]) for full semantics.

### 29.7 Code Generation [over.reflect.codegen]

[1] The following built-in function inserts generated code at the point of the call:

- `mixin(` *expr* `)` — converts a `[char, N]` string expression to a sequence of GAL tokens and inserts those tokens at the point of the call. The resulting token sequence shall be syntactically valid in the context of the insertion point.

[2] *Example:*
```
type Point = object { x*: intd, y*: intd, };

let s     = sizeof<Point>();
let s2    = sizeof(Point(x: 0, y: 0));
let name  = identof(s);           // "s"
let field = fieldof<Point>(0);    // yields Point.x
let n     = lenof<Point>();       // 2

// zeroing all fields via template for
template for i, [0 .. lenof<Point>() - 1] {
    fieldof<Point>(i) = 0;
}

mixin("let x = 1;");
```

---

## 30 Value Categories [expr.cat]

### 30.1 General [expr.cat.general]

[1] Every expression in a GAL program belongs to exactly one value category: either *lvalue* or *rvalue*.

### 30.2 Lvalues [expr.cat.lvalue]

[1] An *lvalue* is an expression that denotes a named, addressable storage location. The following expressions are lvalues:

- A named `let` or `var` binding whose type is not `val T` ([dcl.var]).
- A field of an object, if the object expression is itself an lvalue ([class.general]).
- A pointer subscript expression `p[N]` ([basic.compound.ptr]).
- A function call whose return type is `ref T` ([dcl.fct.general]).

[2] A `const` binding is not an lvalue. It shall not be used as the operand of `addrof` ([basic.compound.addr]).

### 30.3 Rvalues [expr.cat.rvalue]

[1] An *rvalue* is an expression that does not denote an addressable location. The following expressions are rvalues:

- Literals ([lex.literal]).
- The result of an arithmetic, logical, or comparison expression.
- A function call whose return type is not `ref T`.
- A type-constructor expression ([dcl.fct.typecast]).
- A cast expression ([expr.cast]).
- A named binding of type `val T` ([basic.compound.val]).

---

## 31 Name Lookup [basic.lookup]

### 31.1 General [basic.lookup.general]

[1] Name lookup associates a use of an identifier with its declaration. If no declaration is found, the program is ill-formed. If two declarations of the same name are found at the same scope level and do not form an overload set, the program is ill-formed.

[2] A name shall be declared before its first use within the same translation unit. Mutual recursion and other forward references require an explicit forward declaration ([dcl.fwd]).

### 31.2 Unqualified Lookup [basic.lookup.unqual]

[1] For an unqualified name, the compiler searches scopes in the following order, stopping at the first scope in which a declaration is found:

1. The current block scope.
2. Each enclosing block scope, from innermost to outermost.
3. The translation unit scope.

[2] A name imported via `import` ([dcl.import.import]) is visible at translation unit scope. A name introduced via `include` ([dcl.import.include]) is visible as if declared in the translation unit.

### 31.3 Qualified Lookup [basic.lookup.qual]

[1] A qualified name of the form `S::N` looks up `N` directly within scope `S`. If `N` is not found in `S`, the program is ill-formed.

[2] If two separately imported translation units both export the same name and both are visible at the point of use, the program is ill-formed.

### 31.4 Function Call Lookup [basic.lookup.call]

[1] For a UFCS call `x.f(args)` ([over.call]), the compiler first checks whether `f` is a field of the type of `x`. If found, it is treated as a field access. Otherwise, unqualified lookup is performed for a free function `f` whose first parameter type is compatible with the type of `x`. If both a field and a free function are found and the call is ambiguous, the program is ill-formed ([over.call]).

---

## 32 Expressions [expr.types]

### 32.1 Mixed-Type Arithmetic [expr.types.mixed]

[1] Implicit promotion occurs only between types within the same signedness family. Signed integral types (`intb`, `intw`, `intd`, `intq`, `into`, `ssize`) promote among themselves. Unsigned integral types (`uintb`, `uintw`, `uintd`, `uintq`, `uinto`, `usize`) promote among themselves. Cross-signedness arithmetic requires explicit conversion via type-as-constructor ([dcl.fct.typecast]) or `cast` ([expr.cast]).

[2] In a binary arithmetic expression where both operands belong to the same signedness family and differ in width, the operand of the narrower type is implicitly promoted to the wider type before the operation is performed. The result has the wider type.

[3] The width ordering for signed integral types from narrowest to widest is: `intb` < `intw` < `intd` < `intq` < `into`. `ssize` is treated as having the same width as `usize` for promotion ordering purposes and participates in signed promotion accordingly.

[4] The width ordering for unsigned integral types from narrowest to widest is: `uintb` < `uintw` < `uintd` < `uintq` < `uinto`. `usize` participates in unsigned promotion accordingly.

[5] A floating-point type is considered wider than any integral type of equal or lesser abstract size. The width ordering for floating-point types is: `floatw` < `floatd` < `floatq` < `floato`. Promotion between floating-point and integral types requires explicit conversion.

[6] `bool` and `char` are not subject to implicit promotion. Arithmetic expressions involving these types and any other type shall use explicit type-as-constructor conversion or `cast`.

[7] *Example:*
```
let a: intb = 1;
let b: intd = 2;
let c = a + b;          // c has type intd; intb promoted to intd

let d = intd('a') + 1;  // explicit conversion required for char

let e: uintb = 1;
let f: intd  = 2;
// let g = e + f;       // ill-formed: cross-signedness, explicit conversion required
let g = intd(e) + f;    // valid
```

### 32.2 Constant Expressions [expr.types.const]

[1] A *constant expression* is an expression that the compiler can fully evaluate at compile time. The following are ill-formed in a constant expression context:

- `asm` statements ([stmt.asm]).
- `goto` and `label` ([stmt.label]).
- `addrof` applied to any operand ([basic.compound.addr]).
- Any undefined behavior ([intro.undef]).
- Pointer arithmetic where the pointer was not created within the same `const` context.

[2] A function call in a constant expression context is evaluated at compile time if the function body is fully evaluable at compile time, regardless of whether the function is declared `const fn`. If the body cannot be fully evaluated at compile time, the program is ill-formed.

[3] `mixin` ([over.reflect.codegen]) is legal in a constant expression context.

### 32.3 Integer Arithmetic [expr.types.int]

[1] Signed integer overflow is undefined behavior ([intro.undef]).

[2] Unsigned integer arithmetic wraps modulo 2^N where N is the bit width of the type.
