# GAL Grammar Reference

All formal grammar productions from the GAL specification, consolidated for quick reference. Section anchors are provided for each production group.

Conventions:
- *Nonterminals* are in italics in the spec; shown here in plain text with angle brackets e.g. `<type>`
- Terminals are shown in `backticks`
- `_opt` denotes an optional element
- `one of` lists mutually exclusive terminal alternatives

---

## Lexical [lex]

### Keywords [lex.key]
```
addrof   alias    alignof  asm      auto     bool     break    case
cast     char     const    continue elif     else     enum     fieldof
fn       for      goto     if       imut     import   include  identof
label    lenof    let      mixin    mut      object   offsetof ptr
ref      return   sizeof   stringof template this     This     type
typeof   val      var      void
```

### Operators and Punctuators [lex.operators]
```
{   }   [   ]   (   )   ::  .   ->  *
+   -   /   %   =   +=  -=  /=  %=  ==  !=
<   >   <=  >=  &&  ||  !   |   ..  ,   :   ;   _
```

### Escape Sequences [lex.literal]
```
\\   \"   \'   \0
```

---

## Types [basic.types]

```
type:
    fundamental-type
    pointer-type
    reference-type
    value-type
    array-type
    tuple-type
    union-type
    object-type
    enum-type
    auto
    _
    void
    identifier

fundamental-type: one of
    intb  intw  intd  intq  into
    uintb uintw uintd uintq uinto
    usize ssize
    floatw floatd floatq floato
    bool  char
```

---

## Type Qualifiers [basic.type.qualifier]

```
type-qualifier: one of
    mut  imut
```

---

## Declarations [dcl]

### Variable Declaration [dcl.var]
```
variable-declaration:
    declaration-keyword identifier visibility-modifier_opt generic-parameter-list_opt pragma_opt type-annotation_opt initializer_opt ;

declaration-keyword: one of
    const  let  var

type-annotation:
    : type-qualifier_opt type

initializer:
    = expression
```

### Type Declaration [dcl.type]
```
type-declaration:
    type identifier visibility-modifier_opt generic-parameter-list_opt pragma_opt = type ;
```

---

## Wildcard and Auto [dcl.wild]

```
(no grammar productions — contextual uses of _ and auto)
```

---

## Tuples [basic.tuple]

```
tuple-type:
    ( type-list )

type-list:
    type
    type , type-list

tuple-expression:
    ( expression-list )

tuple-destructuring:
    declaration-keyword ( binding-list ) type-annotation_opt = expression ;

binding-list:
    binding
    binding , binding-list

binding:
    identifier
    _
```

---

## Arrays and Ranges [basic.array]

```
array-type:
    type [ array-size ]

array-size:
    expression
    _

range-expression:
    expression .. expression
```

---

## Pointers, References, and Values [basic.compound]

```
pointer-type:
    ptr type-qualifier_opt type

addrof-expression:
    addrof ( expression )

reference-type:
    ref type-qualifier_opt type

value-type:
    val type-qualifier_opt type
```

---

## Unions [basic.union]

```
union-type:
    type | type
    type | union-type
```

---

## Objects [class]

```
object-type:
    object inheritance-list_opt { field-seq_opt }

inheritance-list:
    , identifier
    , identifier inheritance-list

field-seq:
    field
    field field-seq

field:
    declaration-keyword_opt identifier visibility-modifier_opt pragma_opt type-annotation_opt initializer_opt ;
```

---

## Enumerations [dcl.enum]

```
enum-type:
    enum { variant-seq }

variant-seq:
    variant
    variant variant-seq

variant:
    identifier = expression ;
```

---

## Functions and Generics [dcl.fct]

```
function-declaration:
    eval-prefix_opt fn identifier visibility-modifier_opt generic-parameter-list_opt ( parameter-list_opt ) pragma_opt return-type_opt compound-statement

eval-prefix: one of
    const  template

generic-parameter-list:
    < generic-parameter-seq >

generic-parameter-seq:
    identifier
    identifier , generic-parameter-seq

generic-parameter:
    identifier
    identifier = type
    identifier ...

parameter-list:
    parameter
    parameter , parameter-list

parameter:
    declaration-keyword_opt identifier type-annotation_opt initializer_opt

return-type:
    : type

return-statement:
    return expression_opt ;

lambda-expression:
    fn generic-parameter-list_opt ( parameter-list_opt ) return-type_opt compound-statement

type-constructor-expression:
    type ( expression )
```

---

## Templates [dcl.template]

```
template-call:
    identifier ( template-arg-list_opt ) ;
    identifier template-arg-list ;
    expression . identifier ( template-arg-list_opt ) ;
    expression . identifier template-arg-list ;

template-call-with-block:
    identifier ( template-arg-list_opt ) { raw-string-content } ;
    identifier template-arg-list_opt { raw-string-content } ;
    expression . identifier ( template-arg-list_opt ) { raw-string-content } ;
    expression . identifier template-arg-list_opt { raw-string-content } ;

template-arg-list:
    template-arg
    template-arg , template-arg-list

template-arg:
    expression
    identifier
```

---

## Mixins [dcl.mixin]

```
mixin-statement:
    mixin ( expression ) ;
```

---

## Control Flow [stmt]

### General [stmt.general]
```
statement:
    expression-statement
    compound-statement
    variable-declaration
    if-statement
    while-statement
    for-statement
    case-statement
    label-declaration
    goto-statement
    mixin-statement
    return-statement
    break-statement
    continue-statement

compound-statement:
    { statement-seq_opt }

statement-seq:
    statement
    statement statement-seq

break-statement:
    break ;

continue-statement:
    continue ;
```

### Selection Statements [stmt.if]
```
if-statement:
    if-prefix_opt if ( expression ) compound-statement
    if-prefix_opt if ( expression ) compound-statement elif-chain
    if-prefix_opt if ( expression ) compound-statement elif-chain_opt else compound-statement
    if const compound-statement

if-prefix: one of
    const  template

elif-chain:
    elif ( expression ) compound-statement
    elif ( expression ) compound-statement elif-chain
```

### Iteration Statements [stmt.iter]
```
while-statement:
    iter-prefix_opt while expression compound-statement
    iter-prefix_opt while for-init , expression , expression compound-statement

for-statement:
    iter-prefix_opt for for-index_opt element-binding , expression compound-statement

iter-prefix: one of
    const  template

for-init:
    declaration-keyword_opt identifier type-annotation_opt = expression

for-index:
    declaration-keyword_opt identifier type-annotation_opt = expression ,
    _ ,

element-binding:
    declaration-keyword_opt identifier type-annotation_opt
    _
```

### Labels and Goto [stmt.label]
```
label-declaration:
    label identifier ;

goto-statement:
    goto identifier ;
    goto integer-literal ;
```

### Pattern Matching [stmt.case]
```
case-statement:
    case ( expression ) { case-body }

case-body:
    case-label-seq_opt

case-label-seq:
    case-label
    case-label case-label-seq

case-label:
    label case-label-expr : statement-seq_opt

case-label-expr:
    expression
    range-expression
    _
```

---

## Specialized Symbols [over]

```
operator-overload:
    fn ` operator-symbol ` generic-parameter-list_opt ( parameter-list ) return-type_opt compound-statement
```

---

## Function Call Syntax [over.call]

```
call:
    identifier ( argument-list_opt ) ;
    identifier argument-list ;
    expression . identifier generic-parameter-list_opt ( argument-list_opt ) ;
    expression . identifier generic-parameter-list_opt argument-list ;

argument-list:
    expression
    expression , argument-list
```

---

## Pragmas [dcl.pragma]

```
pragma:
    [. pragma-list .]

pragma-list:
    pragma-item
    pragma-item , pragma-list

pragma-item:
    pragma-name
    pragma-name ( expression )
    pragma-name : specifier
    pragma-name : specifier ( expression )
```

---

## Inline Assembly [stmt.asm]

```
asm-statement:
    asm ( asm-string , asm-operand-list , asm-operand-list , asm-operand-list ) ;

asm-string:
    expression

asm-operand-list:
    _
    asm-operand
    asm-operand , asm-operand-list

asm-operand:
    expression
```

---

## Casts [expr.cast]

```
cast-expression:
    cast < type > ( expression )
```

---

## Aliases [basic.scope]

```
alias-declaration:
    alias identifier visibility-modifier_opt = alias-target ;

alias-target:
    expression
    scope-expression

scope-expression:
    { statement-seq_opt }
```

---

## Import and Include [dcl.import]

```
include-directive:
    include string-literal ;

import-directive:
    import string-literal visibility-modifier_opt ;
```
