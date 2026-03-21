# How to Read the GAL Specification

This document explains the conventions used in the GAL — General Applicative Language specification.

---

## Normative vs Non-Normative Content

The specification contains two kinds of content:

**Normative** content defines the language. Conforming implementations and programs must follow it. All numbered paragraphs are normative unless explicitly marked otherwise.

**Non-normative** content provides supplementary explanation. It does not impose requirements. Non-normative content is marked explicitly:

- *Note N: ... — end note* — explanatory remarks that do not add requirements.
- *Example N: ... — end example* — illustrative code that does not define behavior.

---

## The Word "Shall"

**shall** — imposes a normative requirement. A program that violates a "shall" constraint is *ill-formed*.

**shall not** — imposes a normative prohibition. A program that violates it is *ill-formed*.

**may** — denotes permission. The construct is allowed but not required.

**is** / **are** — defines a term or property. Not a requirement, a statement of fact.

---

## Ill-Formed Programs

A program is **ill-formed** when it violates a normative requirement. The compiler is not required to produce a useful error message — it may simply reject the program. Some ill-formed constructs are designated *no diagnostic required*, meaning the compiler is not obligated to detect them at all.

---

## Undefined Behavior

**Undefined behavior** means the specification imposes no requirements on what happens. A conforming implementation may do anything — crash, produce wrong output, appear to work correctly, or format your hard drive. Programs that exhibit undefined behavior are not valid GAL programs even if a particular compiler accepts them.

---

## Implementation-Defined Behavior

**Implementation-defined behavior** is behavior that varies between conforming implementations but must be documented by each implementation. For example, the bit width of abstract size units (`b`, `w`, `d`, `q`, `o`) is implementation-defined.

---

## How to Read BNF Grammar

The specification uses a BNF-style grammar notation. The conventions are:

- *Nonterminals* are written in italics: *type*, *expression*, *identifier*
- `Terminals` are written in monospace backticks: `fn`, `let`, `;`
- A production rule has the form:

> *nonterminal:*
> &nbsp;&nbsp; *alternative-1*
> &nbsp;&nbsp; *alternative-2*

Each indented line is a separate alternative. The nonterminal may expand to any one of them.

- **`_opt`** — the subscript `_opt` on a nonterminal means it is optional. It may appear zero or one times.

> *variable-declaration:*
> &nbsp;&nbsp; *declaration-keyword* *identifier* *visibility-modifier*_opt ...

This means *visibility-modifier* may be present or absent.

- **`one of`** — lists terminals that are alternatives at the same level:

> *declaration-keyword:* one of
> &nbsp;&nbsp; `const` `let` `var`

This means *declaration-keyword* expands to exactly one of `const`, `let`, or `var`.

---

## Section Anchors

Every section has a stable anchor in square brackets, e.g. `[dcl.var]`, `[basic.types]`. Cross-references within the spec use these anchors: `([dcl.var])` means "see section 7.1 Variable Declarations".

---

## Paragraph Numbering

Paragraphs within each section are numbered `[1]`, `[2]`, `[3]`, etc. When referencing a specific paragraph, use the section anchor and paragraph number together, e.g. §7.1 [3].
