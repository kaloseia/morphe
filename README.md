# Morphe Specification (KAMO1) - Unified Application Data Modeling

## Table of Contents

- [Morphe - Application Data Modeling Specification](#morphe---application-data-modeling-specification)
  - [Overview](#overview)
  - [Specification Versions](#specification-versions)
  - [Key Features](#key-features)
  - [Motivation](#motivation)
  - [Field Types](#field-types)
    - [Atomic Field Types](#atomic-field-types)
    - [Enumeration Field Types](#enumeration-field-types)
  - [Structures](#structures)
  - [Models](#models)
    - [Model Fields](#model-fields)
    - [Identifiers](#identifiers)
    - [Related](#related)
      - [Supported Ownership Values](#supported-ownership-values)
      - [Supported Cardinality Values](#supported-cardinality-values)
  - [Entities](#entities)
    - [Entity Fields](#entity-fields)
      - [Indirected Types](#indirected-types)
    - [Entity Identifiers](#entity-identifiers)
    - [Related](#related-1)
      - [Supported Ownership Values](#supported-ownership-values-1)
      - [Supported Cardinality Values](#supported-cardinality-values-1)
  - [Contributing](#contributing)
  - [License](#license)

## Overview

Morphe (KAMO1) is a simple, human-readable base data modeling specification designed to facilitate consistent transpilation of models into multiple programming languages.

The name represents the ancient Greek "form" or "shape", implying an ideal form or prototype from which other forms are derived. This symbolizes how declaratively modelled data in YAML is generatively transformed into code.

## Specification Versions

Version | Status | Description | Repo
--------|---------|------------|------
KAMO1 | 🚧 In Progress | First stable release of the Morphe core specification | [kaloseia/morphe-spec](https://github.com/kaloseia/morphe-spec)
KAMO1-TS1 | 🚧 In Progress | TypeScript transpilation standard for KAMO1 | [kaloseia/morphe-ts-spec](https://github.com/kaloseia/morphe-ts-spec)
KAMO1-GO1 | 🚧 In Progress | Go transpilation standard for KAMO1 | [kaloseia/morphe-go-spec](https://github.com/kaloseia/morphe-go-spec)

## Key Features

1. 📖 **Single Source of Truth**
   * Define your data models once in YAML
   * Automatic code generation across your stack
   * Consistent type definitions across languages
   * Reduced manual synchronization effort

2. 🔄 **Rich Type System**
   * Built-in primitive types (UUID, String, Integer, etc.)
   * Custom enumeration support
   * Relationship modeling (HasOne, HasMany, etc.)
   * Support for protected and sealed fields

3. 🔧 **Flexible Architecture**
   * `Models` for persisted data structures
   * `Structures` for reusable field groups
   * `Entities` for business data aggregation
   * `Enums` for constant value sets
   * Clear separation of concerns

4. 📏 **Standardized Versioning**
   * Core spec versioning (KAMO<x>)
   * Language-specific standards (KAMO<x>-LANG<y>)
   * Backward compatibility guidelines
   * Clear upgrade paths

## Motivation

As your application's codebase grows, structural changes become increasingly expensive. 

Let's take something as simple as adding a `country` field to addresses managed by your application. To achieve this across the stack, you typically need to manually update the following things:

* `address` SQL table creation scripts
* `address` SQL queries
* Backend type definitions, such as an `Address` class
* Associated automated test arrangement and assertion blocks
* API endpoint documentation
* Front-end type definitions, such as an `Address` typescript type
* Front-end form components
* ...

Tooling exists to reduce the manual work needed for some of these steps. Common examples might be an ORM that generates SQL scripts automatically from backend types or GraphQL typescript type generators from GraphQL query files. But these tools are fragmented and tend to leverage multiple disparate "sources of truth" across your application.

The Morphe specification aims to provide a simple schema for automatically updating your code artifacts across technologies, eliminating human error, and accelerating development velocity. 

By consolidating your application's data structures into a single source of truth for your entire stack, you only have to touch one place instead of many and let your selected plugins handle the rest.

## Field Types

### Atomic Field Types

Atomic field types are type primitives that represent a single, indivisible unit of data required for defining fields on higher-order data structures.

* `UUID`: A RFC-4122 compatible UUID string
* `AutoIncrement`: A classic numeric, auto-incrementable record ID
* `String`: A variable-length string
* `Integer`: A numeric value for zero, whole numbers, and their negative counterparts
* `Float`: A numeric floating-point decimal value
* `Boolean`: A boolean (true / false) value
* `Time`: A timestamp value with a UTC offset
* `Date`: A timestamp value as a date with a UTC offset and zero time values
* `Protected`: An encryptable and decryptable value for sensitive information such as API keys
* `Sealed`: A hashable value that can not be decrypted (typically passwords)

### Enumeration Field Types

Enums are predefined sets of constant values that can be used as types for fields within models or structures. They enforce consistency by limiting possible values to a fixed set.

Each enum consists of a name, a primitive type (`String`, `Integer`, `Float`), and a set of entries. Entries are defined as symbol names with an associated primitive representation.

*Example:* `color.enum`

```yaml
name: Color
type: String
entries:
  Red: 'red'
  Blue: 'blue'
  Green: 'green'
```

## Structures

Structures represent standalone, reusable, non-persisted groupings of fields, for more flexible use inside of application projects, for example as subtypes or DTOs (data-transfer-objects).

Each structure consists of a `name`, and a set of `fields`.

**Note:** Unlike models, structures do not have relationships or identifiers, as they are not persisted.

*Example:* `address.str`
```yaml
name: Address
fields:
  Street: 
    type: String
  HouseNr:
    type: String
  ZipCode:
    type: String
  City:
    type: String
```

## Models

Models are the primary persisted data structures. They are somewhat analogous to relational SQL tables.

Each model consists of a `name`, a set of `fields`, a set of `identifiers`, and `related` models.

*Example:* `address.mod`
```yaml
name: Address
fields:
  UUID:
    type: UUID
    attributes:
      - immutable
  Street:
    type: String
  HouseNr:
    type: String
  ZipCode:
    type: String
  City:
    type: String
identifiers:
  primary: UUID
  addr:
    - Street
    - HouseNr
    - ZipCode
    - City
related:
  User:
    type: ForOne
  Document:
    type: HasMany
```

### Model Fields

Denoted by the `fields:` key, the model fields are specified as a uniquely named key and the finer-grained field configuration.

Each field has exactly one type.

Each field may have a list of unconstrained, lower, snake-case attributes. Attributes have no inherent meaning to the Morphe specification but may be required by specific transpiling implementations.

### Identifiers

Denoted by the `identifiers:` key, this section always requires a `primary:` identifier with a corresponding field name.

Composite or secondary identifier groups can be added with a unique key (i.e. `addr:`), and a list of relevant fields.

### Related

Denoted by the `related:` key, this section lists all related models. This means both dependencies and dependents (prerequisites).

Each relationship is characterized by an ownership and a cardinality that differentiates between 1 and n related entities.

#### Supported Ownership Values

* `For`: The current model is "for" related models, when the current model is a dependent on the existence of related models.
  * *Example:* "Address ForOne User": Address is reliant on the existence of a specified User, because Address explicitly references User identifiers.
* `Has`: The current model "has" related models, when the current model is required by the related models.
  * *Example:* "Address HasMany Document": Address supports the reliance of Documents on the existence of specified Addresses, because Documents explicitly reference Address identifiers.

#### Supported Cardinality Values

* `One`: Relationships have a cardinality of one, when the current model's relationship supports one related model instance.
  * *Example:* "Address ForOne User": Address is related to one user.
* `Many`: Relationships have a cardinality of many, when the current model's relationship supports a variable number of related model instances.
  * *Example:* "Owner HasMany Address": Owner is related to many addresses.

## Entities

Entities are indirect data structures that route internally to model field subsets for business data flattening and aggregation. They are analogous to SQL views, decoupling domain data from underlying technical data structures (Models).

Entities consist of a `name`, a set of `fields`, `identifiers`,and related `entities`. Identifiers and field types are inherited from models.

*Example:* `user.ent`
```yaml
name: User
fields:
  UUID:
    type: User.UUID
    attributes:
      - immutable
  Name:
    type: User.Name
  Street:
    type: User.Address.Street
  HouseNr:
    type: User.Address.HouseNr
  ZipCode:
    type: User.Address.ZipCode
  City:
    type: User.Address.City
identifiers:
  primary: UUID
related:
  Company:
    type: ForOne
```

### Entity Fields

Denoted by the `fields:` key, the entity fields are specified as a uniquely named key (example: `Street:`) and the finer-grained field configuration.

Each field may have a list of unconstrained, lower, snake-case (*Example:* `- immutable`, `- mandatory`) attributes. Attributes may be required by specific transpiling implementations, but have no inherent meaning to the Morphe specification itself.

### Entity Identifiers

Denoted by the `identifiers:` key, this section specifies how the entity can be uniquely identified. Entity identification strategies will be expanded in future versions to support various approaches like virtual/composite keys, hierarchical IDs, and surrogate bridging.

Currently, entity identification supports globally unique IDs from a single referenced model field.

#### Indirected Types

Entity field types are indirected model field paths. The path begins with a root model (*Example:* `type: User.Address.Street` -> `User` as the "root model"), and includes related models, terminating in a field of the last related model name. The field inherits the type from the terminal model.

### Related

Denoted by the `related:` key, this section lists all related entities. This means both dependencies and dependents (prerequisites). 

Each relationship is characterized by an `ownership` and a `cardinality` that differentiates between `1` and `n` related entities.

#### Supported Ownership Values

* `For`: The current entity is dependent on related entities.
  * *Example:* "Address ForOne User": Address is reliant on the existence of a specified User, because Address explicitly references User identifiers.
* `Has`: The current entity is required by related entities.
  * *Example:* "Address HasMany Document": Address supports the reliance of Documents on the existence of specified Addresses, because Documents explicitly reference Address identifiers.

#### Supported Cardinality Values

* `One`: The relationship supports one related entity instance.
  * *Example:* "Address ForOne User": Address is related to one user.
* `Many`: The relationship supports multiple related entity instances.
  * *Example:* "Owner HasMany Address": Owner is related to many addresses.

## Contributing

We welcome contributions to the Morphe specification and its language-specific implementations! Here's how you can help:

### Types of Contributions

1. **Specification Improvements**
   * Clarifying existing documentation
   * Suggesting new features or field types
   * Identifying gaps or inconsistencies
   * Improving examples and use cases

2. **Implementation Feedback**
   * Testing language-specific transpilation standards
   * Reporting bugs or unexpected behavior
   * Suggesting optimizations for generated code
   * Proposing new language implementations

### How to Contribute

1. **Issues First**
   * Start by creating an issue to discuss your proposed changes
   * For bugs, include the specification version (e.g., KAMO1, KAMO1-TS1)
   * For features, explain the use case and expected benefits

2. **Pull Requests**
   * Fork the relevant repository
   * Create a branch with a descriptive name
   * Make focused, atomic commits with clear messages
   * Reference the related issue in your PR description

### Style Guidelines

1. **Specification Changes**
   * Use clear, concise language
   * Provide examples for new features
   * Maintain consistent formatting with existing docs
   * Follow YAML best practices in examples

2. **Code Contributions**
   * Follow the language-specific style guide
   * Include tests for new features
   * Update documentation as needed
   * Ensure backwards compatibility

### Questions or Need Help?

Open a discussion in the relevant repository's issues section. We aim to respond within a few business days.

## License

This project is licensed under the MIT License.
