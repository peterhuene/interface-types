# Interface Types Binary Format

## Convention

```
T? ::= 0x00     => ϵ
     | 0x01 t:T => t
```

## Interface Types

Extending [`deftype`](../module-linking/Binary.md#import-definitions) and [`type`](../module-linking/Binary.md#type-definitions):
as defined by module-linking:
```
deftype            ::= ...
                     | 0x06 i:<typeidx>                     => type-index-space[i]
                                                               (must be <adapter-functype>)
type               ::= ...
                     | aft:<adapter-functype>               => aft
                     | cit:<compound-intertype>             => cit
adapter-functype   ::= 0x7c p*:<maybe-named-types>
                            r*:<maybe-named-types>          => (adapter func (param p)* (result r)*)
maybe-named-types  ::= 0x00 t*:vec(<intertype>)             => t*
                     | 0x01 nt*:vec(<named-type>)           => nt*
named-type         ::= n:<name> t:<intertype>               => n t
compound-intertype ::= 0x7b t:<intertype>                   => (list t)
                     | 0x7a nt*:vec(<named-type>)           => (record (field nt)*)
                     | 0x79 nmt*:vec(<named-maybe-type>)    => (variant (case nmt)*)
                     | 0x78 t*:vec(<intertype>)             => (tuple t*)
                     | 0x77 name*:vec(<name>)               => (flags name*)
                     | 0x76 tag*:vec(<name>)                => (enum tag*)
                     | 0x75 t*:vec(<intertype>)             => (union t*)
                     | 0x74 t:<intertype>                   => (optional t)
                     | 0x73 t?:<intertype>? u?:<intertype>? => (expected t? (error u)?)
                     | 0x72 n:<name> t:<intertype>          => (named n t)
named-maybe-type   ::= n:<name> t?:<intertype>              => n t?
intertype          ::= i:<typeidx>                          => type-index-space[i]
                                                               (must be <compound-intertype>)
                     | 0x71                                 => bool
                     | 0x70                                 => s8
                     | 0x6f                                 => u8
                     | 0x6e                                 => s16
                     | 0x6d                                 => u16
                     | 0x6c                                 => s32
                     | 0x6b                                 => u32
                     | 0x6a                                 => s64
                     | 0x69                                 => u64
                     | 0x68                                 => float32
                     | 0x67                                 => float64
                     | 0x66                                 => char
                     | 0x65                                 => string
```


## Canonical Adapter Function Definitions

Extending [`section` as defined by module-linking](../module-linking/Binary.md#module-definitions):
```
section           ::= ...
                    | f*:section_7(vec(<func>))                       => f*
                    | f*:section_8(vec(<adapter-func>))               => f*
func              ::= i:<typeidx> body:<func-body>                    => (func type-index-space[i] body)
                                                                         where i refers to a <core:functype>
func-body         ::= 0x00 opts*:vec(<canon-opt>) f:<adapter-funcidx> => (canon.lower opts* $f)
adapter-func      ::= i:<typeidx> body:<adapter-func-body>            => (adapter func type-index-space[i] body)
                                                                         where i refers to an <adapter-functype>
adapter-func-body ::= 0x00 opts*:vec(<canon-opt>) f:<funcidx>         => (canon.lift opts* $f)
canon-opt         ::= 0x00                                            => string=utf8
                    | 0x01                                            => string=utf16
                    | 0x02                                            => string=compact-utf16
                    | 0x03 f:<funcidx>                                => (with.realloc $f)
                    | 0x04 f:<funcidx>                                => (with.free $f)
```


## Component Definitions

Reusing [top-level productions defined by module-linking](../module-linking/Binary.md#module-definitions):
```
component-version  ::= 0x0a 0x00 0x02 0x00
component-preamble ::= <magic> <component-version>
component          ::= <component-preamble> s*:<section>* => (component flatten(s*))
```
