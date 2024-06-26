# Morphe - Application Data Modelling Specification

v0.0.2
## Table of Contents

- [Introduction](#introduction)
- [Models](#models)
  - [Model Fields](#model-fields)
    - [Atomic Types](#atomic-types)
  - [Identifiers](#identifiers)
  - [Related](#related)
    - [Supported Ownership Values](#supported-ownership-values)
    - [Supported Cardinality Values](#supported-cardinality-values)
- [Entities](#entities)
  - [Entity Fields](#entity-fields)
    - [Indirected Types](#indirected-types)
  - [Related](#related)
    - [Supported Ownership Values](#supported-ownership-values)
    - [Supported Cardinality Values](#supported-cardinality-values)

## Introduction

`Morphe` is a simple, human-readable base data modelling specification.

The name represents the ancient Greek "form" or "shape", implying an ideal form or prototype from which other forms are derived. This symbolizes how declaratively modelled data in YAML is generatively transformed into machine code.

The primary goal is the creation of a centralized, declarative data modeling format that can be utilized by both technical and non-technical stakeholders to specify standardized business application data and rules, that can then be generatively transpiled into layer-specific technologies across the stack.

## Models

Models are the core data structure. They are analogous to SQL tables.

Each model consists of a `name`, a set of `fields`, a set of `identifiers`, and `related` models.

*Simple Example:* `address.mod`
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

Each field may have a list of unconstrained, lower, snake-case (*Example:* "immutable_identifier") attributes. Attributes may be required by specific transpiling implementations, but have no inherent meaning to the Morphe specification itself.

#### Atomic Types

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

*Disclaimer:* Entities are a work in progress and are likely to change more frequently than Models.

Entities represent indirect data structures that route internally to different Model field subsets for business data flattening and aggregation. They are analogous to SQL views. The motivation for this is that it decouples domain data structures from underlying technical data structures (Models) for an improved developer experience and allowing for the simpler specification of business rules / constraints.

Each model consists of a `name`, a set of `fields`, and `related` entities. 

`identifiers` and primitive field types are inherited from the root Model definition (analogous to SQL views). With Entities the primary identifier should be immutable on the Model level (as opposed to Models themselves) to allow for static data migration between different technical services or versions. It is recommended to use `AutoIncrement` ID fields for the model levels, and `UUID` ID fields for the entity levels for improved clarity.

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

All entity field types are indirected model field paths. This means that the path must begin with a singular root model (*Example:* `type: User.Address.Street` -> `User` as the "root model"), then include the related model names or aliases, and terminate in a field of the last related model name. The type is inherited from the terminal model field's type.

As you may suspect, "many" related model relationships are problematic for indirected types, and are still unsupported until we have explored the "Entity to Model" space more.

### Related

Denoted by the `related:` key, this section lists all related entities. This means both dependencies and dependents (prerequisites). 

This is an atypical design decision, made for full system transparency and flexibility, with the assumption that configuration errors are easily detectable with the correct tooling.

Each relationship is characterized by an `ownership`, as in the directionality of the relationship, and a `cardinality` that differentiates between `1` and `n` related entities. 

Polymorphism is not supported yet on the entity level due to dealing with unknown trade-offs.

*Note:* Related entity names and indirected field types may contain the same names, which can be confusing. This is still valid, but likely a code-smell since related models are generally self-contained on the entity level.  

#### Supported Ownership Values

* `For`: The current entity is "for" related entities, when the current entity is a dependent on the existence of related entities.
  * *Example:* "Address ForOne User": Address is reliant on the existence of a specified User, because Address explicitly references User identifiers.
* `Has`: The current entity "has" related entities, when the current entity is required by the related entities.
  * *Example:* "Address HasMany Document": Address supports the reliance of Documents on the existence of specified Addresses, because Documents explicitly reference Address identifiers.

#### Supported Cardinality Values

* `One`: Relationships have a cardinality of one, when the current model's relationship supports one related model instance.
  * *Example:* "Address ForOne User": Address is related to one user.
* `Many`: Relationships have a cardinality of many, when the current model's relationship supports a variable number of related model instances.
  * *Example:* "Owner HasMany Address": Owner is related to many addresses.
