# xsd_parser TypeAlreadyDefined Bug - Minimal Reproduction

I ran into this error when trying to parse NeTEx XSD schemas:

```
Error: InterpreterError(TypeAlreadyDefined(Ident {
  ns: Some(NamespaceId(2)),
  name: Generated("BaseMyGroup"),
  type_: Type
}))
```

## The Problem

When two complexTypes have names that differ only by a trailing underscore (like `Base_` and `Base`), and both reference the same `xsd:group`, the parser generates duplicate type names.

## Root Cause

The xsd_parser strips trailing underscores when generating nested type names for group references:

- `Base_` + `MyGroup` → `BaseMyGroup`
- `Base` + `MyGroup` → `BaseMyGroup` ← **collision!**

## How to Reproduce

1. Point `build.rs` to this XSD:

   ```rust
   config.parser.schemas = vec![Schema::File("./minimal_reproduction_schema.xsd".into())];
   ```

2. Run `cargo build`

## Schema Modes

I've set this up so you can switch between the minimal reproduction and the real NeTEx schemas using an environment variable:

```bash
# Default - minimal reproduction schema
cargo build

# Real NeTEx XSD files (you'll need the NeTEx repo at ./NeTEx)
XSD_MODE=netex cargo build
```

## Why This Matters

This pattern is common in NeTEx XSD schemas. Abstract base types often have a trailing underscore (e.g., `Assignment_VersionStructure_`) and concrete types have the same name without it (e.g., `Assignment_VersionStructure`).
