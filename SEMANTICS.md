# GAL Semantic Rules Reference

This document describes the semantic analysis rules for GAL in procedural form, intended as a reference for compiler implementors. Rules are given in the order a single-pass compiler would encounter them.

---

## 1. Name Resolution

### 1.1 Lookup Order
To resolve an unqualified name `N` at a point `P`:
1. Search the current block scope for a declaration of `N` at or before `P`.
2. If not found, search each enclosing block scope from innermost to outermost.
3. If not found, search the translation unit scope (including imported names).
4. If not found, the program is ill-formed: undefined name.

### 1.2 Qualified Lookup
To resolve `S::N`:
1. Resolve `S` as a scope (enum type, alias, or named scope).
2. Search `S` directly for `N`.
3. If not found, the program is ill-formed: `N` is not a member of `S`.

### 1.3 Forward Declarations
A forward declaration introduces a name at the current scope before its full definition. When a full definition is later encountered for a forward-declared name, the compiler shall verify:
- For functions: parameter types and return type match the forward declaration exactly.
- For types: the full definition is a valid complete type.
- For variables: the type matches the forward declaration exactly.

### 1.4 UFCS Lookup
For a call `x.f(args)`:
1. Check if `f` is a field of the type of `x`. If yes, treat as field access.
2. Otherwise, perform unqualified lookup for a free function `f` whose first parameter type is compatible with the type of `x`.
3. If both a field and a free function are found and the call is syntactically ambiguous, the program is ill-formed.

---

## 2. Type Checking

### 2.1 Declaration Type Deduction
For a declaration with no type annotation:
- The type of the binding is the type of the initializer expression.
- If there is no initializer, the program is ill-formed.

For a function parameter with no type annotation:
- The type is deduced from the default argument initializer.
- If there is no default argument, the program is ill-formed.

### 2.2 Assignment Compatibility
An expression `e` of type `T` may be assigned to a binding of type `U` if:
- `T` and `U` are the same type, or
- `T` is promotable to `U` under the promotion rules ([expr.types.mixed]).

Otherwise the program is ill-formed: type mismatch.

### 2.3 Mutability Checking
Before any write to a binding or through a pointer:
1. If the binding is `const`, the write is ill-formed.
2. If the binding is `let` and the type qualifier is `imut` (or inferred `imut`), the write is ill-formed.
3. If the pointer's pointee type qualifier is `imut`, a write through the pointer is ill-formed.
4. Otherwise the write is permitted.

### 2.4 Promotion Rules
Given a binary arithmetic expression with operands of types `L` and `R`:
1. If `L` and `R` are the same type, no promotion occurs.
2. If `L` and `R` are both signed integral types (including `ssize`), the narrower is promoted to the wider.
3. If `L` and `R` are both unsigned integral types (including `usize`), the narrower is promoted to the wider.
4. If `L` and `R` are both floating-point types, the narrower is promoted to the wider.
5. If `L` and `R` are from different signedness families, the program is ill-formed: explicit conversion required.
6. If either is `bool` or `char`, the program is ill-formed: explicit conversion required.

Width ordering (narrowest to widest):
- Signed: `intb` < `intw` < `intd` < `intq` < `into`; `ssize` has width equal to `usize`.
- Unsigned: `uintb` < `uintw` < `uintd` < `uintq` < `uinto`; `usize` has implementation-defined width.
- Float: `floatw` < `floatd` < `floatq` < `floato`.

### 2.5 Enum Type Checking
For a variant construction `E::V(expr)`:
1. Resolve `E` as an enum type.
2. Resolve `V` as a type variant of `E`.
3. Check that `expr` is assignable to the type of variant `V`.
4. The result type is `E`.

For a value variant access `E::V`:
1. Resolve `E` as an enum type.
2. Resolve `V` as a value variant of `E`.
3. The result type is the type of the assigned expression of `V`.

### 2.6 Generic Instantiation
For each call to a generic function `f<T>(args)`:
1. Substitute all generic parameters with the supplied arguments.
2. Perform full type checking on the substituted body.
3. If type checking fails, the program is ill-formed at the call site.
4. If a prior instantiation with the same arguments exists, reuse it.

---

## 3. Scope Rules

### 3.1 Scope Introduction
A new scope is introduced by:
- A compound statement `{ }`.
- A function body.
- A `for` or `while` loop body.
- A `case` arm body.
- An `alias` scope expression.

### 3.2 Name Export
A name declared with `*` inside a compound statement scope is exported to the immediately enclosing scope. It is as if the name were declared in the enclosing scope at the point the compound statement ends.

### 3.3 Shadowing
A name declared in an inner scope may shadow a name in an outer scope. The outer name remains accessible as `_::name`. Shadowing is not an error.

### 3.4 Visibility at Translation Unit Boundary
- `include`: all names in the included file are available as if declared in the including TU.
- `import`: all names in the imported TU are available in the importing TU. Names without `*` are not re-exported by the importing TU. Names with `*` are re-exported only if the `import` directive itself carries `*`.

---

## 4. Lvalue and Rvalue Rules

### 4.1 Determining Value Category
An expression is an lvalue if it is one of:
- A named `let` or `var` binding (not `val T`).
- A field access on an lvalue object.
- A pointer subscript `p[N]`.
- A function call returning `ref T`.

An expression is an rvalue otherwise, including:
- All literals.
- Arithmetic, logical, or comparison results.
- Function calls returning non-`ref` types.
- Type-constructor expressions.
- Cast expressions.
- Bindings of type `val T`.

### 4.2 addrof Requirements
Before processing `addrof(expr)`:
1. Verify `expr` is an lvalue. If not, the program is ill-formed.
2. Verify the binding of `expr` is not `const`. If it is, the program is ill-formed.

---

## 5. Compile-Time Evaluation

### 5.1 const Context
A `const` declaration or `const if`/`const for`/`const while` introduces a compile-time context. In a compile-time context:
- All expressions shall be fully evaluable at compile time.
- `asm`, `goto`, `label`, and `addrof` are ill-formed.
- Function calls are evaluated at compile time if the body is fully evaluable.
- `mixin` is legal and executes at compile time.

### 5.2 Constant Expression Verification
To verify an expression `e` is a constant expression:
1. Recursively verify all subexpressions.
2. Reject if any subexpression is `asm`, `goto`, `label`, or `addrof`.
3. Reject if any subexpression calls a function whose body contains a non-constant subexpression.
4. Reject if any pointer arithmetic involves a pointer not created in the same `const` context.

---

## 6. Pattern Matching Rules

### 6.1 case Operand Type
The type of the `case` operand determines which label forms are valid:
- Any integral type: expression labels and range labels are valid.
- Any enum type: variant pattern labels are valid; expression labels matching discriminant values are also valid.
- Any type: the wildcard label `_` is always valid.

### 6.2 Variant Pattern Binding
For a label `label E::V(id):`:
1. Verify the `case` operand has type `E`.
2. Verify `V` is a type variant of `E`.
3. Introduce `id` into the arm's scope as a `let` binding of the type associated with variant `V`.

For a label `label E::V:`:
1. Verify the `case` operand has type `E`.
2. Verify `V` is a variant of `E` (value or type).
3. If `V` is a value variant, implicitly rebind the `case` operand to the variant's inner value within the arm.
4. No additional binding is introduced.

### 6.3 Exhaustiveness
The compiler should warn (not error) when a `case` on an enum type does not cover all variants and has no wildcard arm. Exhaustiveness is not enforced as an error.

### 6.4 Fall-Through
Fall-through does not occur. After the last statement of a `case` arm's statement-seq, control transfers to the point immediately following the `case` statement. `break` within a `case` arm exits the arm early (same effect as reaching the end of the arm) unless inside an enclosing loop, in which case it exits the loop.

---

## 7. Function Rules

### 7.1 Return Type Deduction
For a function with no declared return type:
1. If the function body contains no `return expr` statement, the return type is `void`.
2. If the function body contains one or more `return expr` statements, the return type is the type of the first such expression.
3. All `return expr` statements shall yield expressions of the same type. If they differ, the program is ill-formed.

### 7.2 Template Functions
A template function body is not type-checked at the point of definition. It is embedded textually at each call site and type-checked in the call site's context.

### 7.3 Generic Functions
A generic function body is type-checked once per unique instantiation. Errors are reported at the call site.

### 7.4 UFCS Resolution
`x.f(args)` is semantically equivalent to `f(x, args)`. The receiver `x` is passed as the first argument. The call is subject to standard overload resolution for `f` with first argument type matching the type of `x`.

---

## 8. Object Rules

### 8.1 Field Initializer Evaluation
Field initializers are evaluated in declaration order each time an object is constructed. A `const` field initializer is evaluated once at type-definition time.

### 8.2 Named Construction
Object construction requires named arguments. The compiler shall:
1. Match each named argument to a field of the object type by name.
2. Evaluate each matched initializer.
3. For fields with no supplied argument, use the field's default initializer if present.
4. If a field has no default initializer and no argument is supplied, the program is ill-formed.

### 8.3 this and This
`this` is available only within an object body. Its type is `ptr EnclosingType`. `This` is available only within an object body. Its value is the enclosing object type. Use of either outside an object body is ill-formed.

---

## 9. Operator Overloading

### 9.1 Overload Resolution
For a binary operator expression `a OP b`:
1. Check if a user-defined overload `` `OP` `` exists with compatible parameter types.
2. If yes, call it.
3. If no overload exists, apply built-in operator semantics if the types are fundamental.
4. If neither applies, the program is ill-formed.

### 9.2 Built-in Operator Restrictions
Built-in operators apply only to fundamental types. `bool` and `char` do not participate in arithmetic operators without explicit conversion.

---

## 10. Import and Include Semantics

### 10.1 include
Processing `include "file"`:
1. Locate the file relative to the current translation unit's directory (implementation-defined search path).
2. Tokenize and parse the file as if its contents were inserted at the point of the directive.
3. All declarations become native to the including TU; visibility modifiers have no special effect at the inclusion boundary.

### 10.2 import
Processing `import "file"`:
1. Locate and process the named translation unit.
2. Make all its declarations available in the current TU.
3. Declarations without `*` are usable in the current TU but shall not be re-exported.
4. Declarations with `*` are re-exported only if this `import` directive carries `*`.
5. If two imported TUs both export the same name and both are visible at the same point, the program is ill-formed.
