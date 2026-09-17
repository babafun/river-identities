# Primitive Registry Structure

This repository uses a lightweight data registry pattern under `registry/` to define reusable identity primitives. A primitive is a small, named, domain-scoped preset that provides a bundle of normalized numeric attributes. It is intentionally not a full identity by itself; instead, it acts as a building block that can be composed into a larger identity profile.

The overall design is extremely simple:

- `registry/` is the root registry namespace.
- Each top-level folder under `registry/` is a domain, such as `gender`.
- Each domain has an `index.*` file that lists the available primitive entries in that domain.
- Each listed primitive resolves to a directory containing a `primitive.*` file.
- The primitive file contains metadata in a fixed `head` section and values in a fixed `body` section.
- The repository stores the same structure in both TOML and JSON to make it human-readable and machine-friendly.

## 1. Root layout

```text
registry/
├── gender/
│   ├── index.toml
│   ├── index.json
│   ├── male/
│   │   ├── primitive.toml
│   │   └── primitive.json
│   ├── female/
│   │   ├── primitive.toml
│   │   └── primitive.json
│   ├── maxmasc/
│   │   ├── primitive.toml
│   │   └── primitive.json
│   └── maxfem/
│       ├── primitive.toml
│       └── primitive.json
└── personality/
    └── .gitkeep
```

This is the current structure of the registry. The `personality` folder exists as a placeholder, which indicates the registry is designed to grow by domain. The `gender` domain is the working example and shows the canonical shape of every registry section.

## 2. Why the registry exists

The README defines a primitive as a foundational trait or preset: a basic building block for an identity. A primitive is not a complete persona. It describes one facet of identity, such as gender expression, in a bounded numeric form.

This means the registry is a dictionary of reusable trait profiles designed to be stitched together later into larger identity objects. In practice, each primitive acts like a normalized parameter bundle:

- a name
- a domain
- a version
- a trait vector or profile

The goal is not to hardcode large identities into individual files; instead, the system creates a collection of reusable, composable trait building blocks.

## 3. The domain index file

Every domain has an `index.toml` and `index.json` file, acting as the directory map for that domain.

Example: `registry/gender/index.toml`

```toml
[head]
version = "0.1.0"
id = "gender"
domain = "index"

[body]
male = "male/primitive.toml"
female = "female/primitive.toml"
maxmasc = "maxmasc/primitive.toml"
maxfem = "maxfem/primitive.toml"
```

Equivalent JSON:

```json
{
    "head": {"version": "0.1.0", "id": "gender", "domain": "index"},
    "body": {
        "male": "male/primitive.json",
        "female": "female/primitive.json",
        "maxmasc": "maxmasc/primitive.json",
        "maxfem": "maxfem/primitive.json"
    }
}
```

This file does three important things:

1. Declares the domain identity
   - `head.id = "gender"`
   - `head.domain = "index"`
   - This tells you the index file belongs to the `gender` domain and is the registry index for that domain.

2. Records the registry version
   - `head.version = "0.1.0"`
   - This is metadata for compatibility and schema evolution.

3. Maps names to primitive files
   - `body.male = "male/primitive.toml"`
   - `body.female = "female/primitive.toml"`
   - etc.
   - In other words, the index is a catalogue: names are keys, file paths are values.

This means a consumer can discover the domain from the top-level folder and then resolve a primitive by name from the index file without needing a separate database or registry service.

## 4. Naming conventions

The registry is intentionally hierarchical and path-based.

### 4.1 Domain name

The folder name is the domain, e.g.:

- `registry/gender/`
- `registry/personality/`

This is the namespace for that family of primitives.

### 4.2 Primitive name

Inside the domain, each primitive has a folder name matching its symbolic name, such as:

- `male`
- `female`
- `maxmasc`
- `maxfem`

The fully qualified identifier is usually assembled as:

- `domain.name`

For example:

- `gender.male`
- `gender.female`
- `gender.maxmasc`
- `gender.maxfem`

This matches the `head.id` field inside each primitive file.

### 4.3 File path resolution

The index file stores relative paths like:

```text
male/primitive.toml
```

That path is resolved relative to the domain folder. So the actual file is:

```text
registry/gender/male/primitive.toml
```

This makes the registry easy to traverse as a static file tree without requiring code generation.

## 5. Primitive file schema

Each primitive file has the same two-part structure:

- `head`: metadata describing the primitive
- `body`: the actual trait values that define the primitive

Example: `registry/gender/female/primitive.toml`

```toml
[head]
version = "1.0.0"
id = "gender.female"
domain = "gender"

[body]
masc = 0.3
fem = 0.95
neut = 0.25
liquid = 0.0
```

Equivalent JSON:

```json
{
    "head": {
        "version": "1.0.0",
        "id": "gender.female",
        "domain": "gender"
    },
    "body": {
        "masc": 0.3,
        "fem": 0.95,
        "neut": 0.25,
        "liquid": 0.0
    }
}
```

### 5.1 `head`

The `head` block is metadata and not the actual content payload.

It contains:

- `version`: schema or primitive version
- `id`: canonical identifier, usually `domain.primitive`
- `domain`: parent namespace

This is the registry identity record for the primitive.

### 5.2 `body`

The `body` block is the actual primitive definition. It is a set of floating-point values, all normalized to a 0.0–1.0 range.

In the current example, the exact fields are:

- `masc`
- `fem`
- `neut`
- `liquid`

These values represent traits in a trait vector rather than a boolean or enum. The numbers act like weights or intensity values:

- `masc = 0.95` means highly masculine-coded
- `fem = 0.95` means highly feminine-coded
- `neut = 0.25` means some neutral element
- `liquid = 0.0` means no liquid/shape-shifting effect in this preset

The important point is that a primitive is not a single label. It is a parameter bundle. Each field contributes to a larger identity model, even though the current repository only stores a very simple representation.

## 6. What the values mean

The current registry values are simple normalized coefficients:

- 0.0 = absent / minimum
- 1.0 = maximal / full expression
- intermediate values represent blending or partial presence

This gives the registry a mathematically composable shape. A downstream system could combine multiple primitive profiles by summing, averaging, or otherwise blending them to produce a composite identity state.

Example from the `gender` domain:

- `male` = `masc: 0.95`, `fem: 0.3`, `neut: 0.25`, `liquid: 0.0`
- `female` = `masc: 0.3`, `fem: 0.95`, `neut: 0.25`, `liquid: 0.0`
- `maxmasc` = `masc: 1.0`, `fem: 0.0`, `neut: 0.0`, `liquid: 0.0`
- `maxfem` = `masc: 0.0`, `fem: 1.0`, `neut: 0.0`, `liquid: 0.0`

These are not arbitrary. They represent a gradient system where the registry can express a spectrum from masculine to feminine and intermediate states. The `maxmasc` and `maxfem` presets are the extreme endpoints of the domain; the `male` and `female` presets are softer, less absolute versions of those extremes.

## 7. The role of JSON and TOML duplication

The repository stores both TOML and JSON versions of each registry record.

This is a common pattern for static data registries because it supports:

- human readability in TOML
- easy machine parsing in JSON
- cross-language interoperability
- simple repository diffing and editing

The files are structurally equivalent. The only difference is serialization format.

For example:

- `registry/gender/index.toml`
- `registry/gender/index.json`

and

- `registry/gender/female/primitive.toml`
- `registry/gender/female/primitive.json`

represent the same data. A loader could choose either format depending on availability or implementation preference.

## 8. How the registry is resolved at runtime

The file layout implies a simple static resolution algorithm:

1. Pick a domain folder, for example `registry/gender/`.
2. Read the domain `index.*` file.
3. Look up the requested primitive name in `body`.
4. Resolve the relative path value to a file such as `male/primitive.toml`.
5. Parse the primitive file.
6. Read the `head.id` and `body` values.
7. Use those values as the trait definition for the identity component.

This is a filesystem-backed registry rather than a database-backed registry. There is no central registry object or runtime service in this repository yet; the registry is simply a folder tree and metadata format.

In other words, the data is discoverable through structure:

```text
registry/<domain>/index.* -> body.<primitive_name> -> <primitive_name>/primitive.*
```

## 9. How to add a new primitive

A new primitive fits the same pattern.

1. Create a new folder under the relevant domain, for example:

```text
registry/gender/nonbinary/
```

2. Create the primitive file pair:

```text
registry/gender/nonbinary/primitive.toml
registry/gender/nonbinary/primitive.json
```

3. Define the metadata:

```toml
[head]
version = "1.0.0"
id = "gender.nonbinary"
domain = "gender"
```

4. Define the trait payload:

```toml
[body]
masc = 0.45
fem = 0.45
neut = 0.9
liquid = 0.0
```

5. Update the domain `index.*` file to include the new entry:

```toml
[body]
nonbinary = "nonbinary/primitive.toml"
```

That is the entire extension model: name the primitive, assign a canonical id, define a trait vector, register it in the domain index.

## 10. Design intent

The primitive registry is intentionally minimal and modular. It is designed around a few conceptual rules:

- domain-based organization
- static file discovery
- machine-readable metadata
- normalized numeric trait values
- composition-friendly primitives
- extensibility without code changes

This keeps the repository suitable for experimentation and future identity composition, while remaining easy to inspect, edit, and version in git.

## 11. Summary

The primitive registry works like a static, file-based trait catalog:

- `registry/` contains named domains.
- each domain has an `index.*` catalogue file.
- each entry in the index points to a primitive definition file.
- each primitive file has a consistent `head` + `body` format.
- each primitive contains a normalized trait vector representing a slice of identity.
- the repository maintains both TOML and JSON formats for the same data.

The `gender` domain is the clearest example and demonstrates the canonical structure that future domains should follow.

If another domain is added later, it should follow the same rule set: create a folder under `registry/`, add an `index.*` file, and define primitive entries with `head.id`, `head.domain`, and a `body` trait profile.

## 12. Current canonical example

The working example in this repository is the `gender` domain:

```text
registry/gender/index.toml
registry/gender/male/primitive.toml
registry/gender/female/primitive.toml
registry/gender/maxmasc/primitive.toml
registry/gender/maxfem/primitive.toml
```

These files encode a domain-scoped trait registry where each primitive is a numeric profile and each name resolves to a trait definition using a simple, filesystem-based registry convention.

That is the primitive registry in its current form: a small, explicit, composable data model for identity traits.
