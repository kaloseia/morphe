# Morphe - Application Data Modeling Specification

v0.0.3

## Table of Contents

- [Morphe - Application Data Modeling Specification](#morphe---application-data-modeling-specification)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
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
    - [Related](#related-1)
      - [Supported Ownership Values](#supported-ownership-values-1)
      - [Supported Cardinality Values](#supported-cardinality-values-1)

## Introduction

`Morphe` is a simple, human-readable base data modeling specification.

The name represents the ancient Greek "form" or "shape", implying an ideal form or prototype from which other forms are derived. This symbolizes how declaratively modelled data in YAML is generatively transformed into machine code.

The primary goal is the creation of a centralized, declarative data modeling format that can be utilized by both technical and non-technical stakeholders to specify standardized business application data and rules, that can then be generatively transpiled into layer-specific technologies across the stack.

## Field Types

- [Morphe - Application Data Modeling Specification](#morphe---application-data-modeling-specification)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
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
    - [Related](#related-1)
      - [Supported Ownership Values](#supported-ownership-values-1)
      - [Supported Cardinality Values](#supported-cardinality-values-1)


### Atomic Field Types

Atomic field types are type primitives that is a single, indivisible unit of data required for defining fields on higher-order data structures.

* `UUID`: A RFC-4122 compatible UUID string.
* `AutoIncrement`: A classic numeric, auto-incrementable record ID.
* `String`: A variable-length string.
* `Integer`: A numeric value for zero, whole numbers, and their negative counterparts.
* `Float`: A numeric floating-point decimal value.
* `Boolean`: A boolean (true / false) value.
* `Time`: A timestamp value with a UTC offset.
* `Date`: A timestamp value as a date with a UTC offset and zero time values.
* `Protected`: An encryptable and decryptable value for sensitive information such as API keys.
* `Sealed`: A hashable value that can not be decrypted (typically passwords).

### Enumeration Field Types

Enums are predefined sets of constant values that can be used as types for fields within models or structures. They enforce consistency by limiting possible values to a fixed set.

Each enum consists of a `name`, and a set of `entries`. Entries are defined as symbol names with an associated primitive representation.

*Example:* `color.enum`
```yaml
name: Color
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

Denoted by the `fields:` key, the model fields are specified as a uniquely named key (example: `Street:`) and the finer-grained field configuration.

Each field has exactly one type.

Each field may have a list of unconstrained, lower, snake-case (*Example:* "immutable_identifier") attributes. Attributes have no inherent meaning to the Morphe specification but may be required by specific transpiling implementations.

### Identifiers

Denoted by the `identifiers:` key, this section always requires a `primary:` identifier with a corresponding field name.

Composite or secondary identifier groups can be added with a unique key (i.e. `addr:`), and a list of relevant fields.

### Related

Denoted by the `related:` key, this section lists all related models. This means both dependencies and dependents (prerequisites). 

This is an atypical design decision, made for full system transparency and flexibility, with the assumption that configuration errors are easily detectable with the correct tooling.

Each relationship is characterized by an `ownership`, as in the directionality of the relationship, a `cardinality` that differentiates between `1` and `n` related entities, and optional `polymorphism` if the related model types are multivariate and unknown.

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

Entities consist of a `name`, a set of `fields`, and related `entities`. Identifiers and field types are inherited from models.

*Simple Example:* `user.ent`
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
related:
  Company:
    type: ForOne
```

### Entity Fields

Denoted by the `fields:` key, the entity fields are specified as a uniquely named key (example: `Street:`) and the finer-grained field configuration.

Each field may have a list of unconstrained, lower, snake-case (*Example:* `- immutable`, `- mandatory`) attributes. Attributes may be required by specific transpiling implementations, but have no inherent meaning to the Morphe specification itself.

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
