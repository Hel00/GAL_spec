# GAL Memory Layout Reference

This document specifies the binary representation of all GAL types in terms of abstract size units. Concrete bit widths of abstract units are implementation-defined ([intro.impldef]) and shall be documented by each conforming implementation. All sizes and alignments below are expressed in abstract units unless otherwise noted.

---

## Abstract Size Units

| Designator | Name |
|---|---|
| `b` | byte |
| `w` | word |
| `d` | double-word |
| `q` | quad-word |
| `o` | octa-word |

The relationship between units is implementation-defined. A conforming implementation shall document the bit width of each unit for its target platform. All that is guaranteed is that the units are ordered by width: a byte is no wider than a word, a word no wider than a double-word, and so on.

---

## Fundamental Types

Each type's size and alignment are expressed in terms of the abstract unit it is named after.

| Type | Size | Alignment | Representation |
|---|---|---|---|
| `intb` | 1b | 1b | signed two's complement |
| `intw` | 1w | 1w | signed two's complement |
| `intd` | 1d | 1d | signed two's complement |
| `intq` | 1q | 1q | signed two's complement |
| `into` | 1o | 1o | signed two's complement |
| `uintb` | 1b | 1b | unsigned |
| `uintw` | 1w | 1w | unsigned |
| `uintd` | 1d | 1d | unsigned |
| `uintq` | 1q | 1q | unsigned |
| `uinto` | 1o | 1o | unsigned |
| `usize` | platform-defined | platform-defined | unsigned; sufficient to represent any object size or pointer |
| `ssize` | same as `usize` | same as `usize` | signed two's complement |
| `floatw` | 1w | 1w | implementation-defined floating-point |
| `floatd` | 1d | 1d | implementation-defined floating-point |
| `floatq` | 1q | 1q | implementation-defined floating-point |
| `floato` | 1o | 1o | implementation-defined floating-point |
| `bool` | 1b | 1b | 0x00 = false, 0x01 = true; all other bit patterns are undefined behavior |
| `char` | implementation-defined | implementation-defined | implementation-defined |
| `void` | 0 | 1b | no storage allocated |

---

## Pointers

A pointer (`ptr T`) has the same size and alignment as `usize`. It holds an absolute memory address. A null pointer is represented as the all-zero bit pattern.

---

## References

A reference (`ref T`) has the same size, alignment, and representation as `ptr T`. A reference is guaranteed non-null; the compiler shall not store a null bit pattern in a reference.

---

## Values

A value type (`val T`) has the same size, alignment, and representation as `T`.

---

## Arrays

An array `[T, N]` is a contiguous sequence of `N` elements of type `T` with no padding between elements. Its total size is `N * sizeof<T>()`. Its alignment is `alignof<T>()`. Element `i` is at byte offset `i * sizeof<T>()` from the base address of the array.

---

## Tuples

A tuple `(T0, T1, ..., Tn)` is laid out as fields in declaration order. Each field is placed at the next offset that satisfies its alignment requirement. Padding bytes are inserted between fields as needed. The tuple's overall alignment is the maximum alignment among its element types. The total size is rounded up to a multiple of the tuple's alignment.

Example layout of `(intb, intd, intw)` — sizes and padding depend on the implementation's unit widths:
```
offset 0:                    intb  (1b)
offset 1b .. alignof(intd):  padding
offset alignof(intd):        intd  (1d)
offset alignof(intd)+1d:     intw  (1w)
trailing padding to round to alignof(intd)
overall alignment: alignof(intd)
```

---

## Objects

An object type is laid out as a struct. Fields are placed in the following order:

1. All fields inherited from base types, recursively, in base-declaration order (leftmost base first), each base's own fields in declaration order.
2. The object's own directly declared fields, in declaration order.

Padding is inserted between fields to satisfy each field's alignment requirement. The object's overall alignment is the maximum alignment of all its fields including inherited ones. The total size is rounded up to a multiple of the object's alignment.

The `[.packed.]` pragma suppresses all padding. In a packed object, fields are placed at consecutive byte offsets with no gaps regardless of alignment.

The `[.bit: set(N).]` pragma constrains a field to occupy exactly N bits within its containing storage unit. Bit fields are packed consecutively within a storage unit of the field's declared type. A bit field shall not span storage unit boundaries.

Example (unit widths are implementation-defined):
```
type Base = object { a: intb, b: intd, };
type Child = object, Base { c: intw, };

// Conceptual layout of Child:
// [inherited from Base]
//   offset 0:                    a (intb, 1b)
//   offset 1b .. alignof(intd):  padding
//   offset alignof(intd):        b (intd, 1d)
// [own fields]
//   offset alignof(intd)+1d:     c (intw, 1w)
//   trailing padding to round to alignof(intd)
// overall alignment: alignof(intd)
```

---

## Enumerations

An enumeration is laid out as a tagged union:

1. A discriminant field of type `uintd` at offset 0, holding the variant's ordinal (0 for the first declared variant, incrementing by 1 for each subsequent variant in declaration order).
2. A payload field at the next offset satisfying the alignment of the largest type variant payload, sized to hold that largest payload. Value variants contribute no payload storage.
3. Trailing padding to round the total size to a multiple of the overall alignment.

The enumeration's overall alignment is the maximum of `alignof<uintd>()` and the alignment of the largest type variant payload.

If the enumeration has no type variants (all variants are value variants), no payload field is present. The size is `sizeof<uintd>()` and the alignment is `alignof<uintd>()`.

Example (unit widths are implementation-defined):
```
type Number = enum {
    Int   = intd,   // discriminant 0, payload: intd
    Float = floatq, // discriminant 1, payload: floatq
};

// Layout:
// offset 0:                         discriminant (uintd)
// offset sizeof(uintd):             padding to align floatq
// offset alignof(floatq):           payload (sized for floatq)
// trailing padding to round to alignof(floatq)
// overall alignment: max(alignof(uintd), alignof(floatq))
```

Value-variant-only example:
```
type Status = enum {
    Ok    = 0,
    Error = 1,
};
// Layout: discriminant only
// size: sizeof(uintd), alignment: alignof(uintd)
```

---

## Functions and Lambdas

A named function declared at translation unit scope occupies no data storage. Its address is the address of its first instruction in the output binary.

A lambda has no implicit closure. It is represented as a function pointer with the same size and alignment as `ptr void`.

---

## `void`

`void` has size 0 and alignment 1b. No storage is allocated for a `void`-typed binding.

---

## Static Members

A field declared with `[.static.]` is not stored within any object instance. It is allocated once at translation unit scope as a single shared instance with the size and alignment of its declared type.
