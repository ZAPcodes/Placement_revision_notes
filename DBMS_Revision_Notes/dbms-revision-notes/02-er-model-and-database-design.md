# 02. ER Model and Database Design

> ER modeling converts requirements into entities, relationships and constraints before choosing tables.

## Core elements

- **Entity:** distinguishable real-world object.
- **Entity set:** collection of similar entities.
- **Attribute:** property of an entity/relationship.
- **Relationship:** association among entities.
- **Key attribute(s):** uniquely identify an entity.

## Attribute types

| Type | Example | Mapping concern |
|---|---|---|
| Simple | salary | One atomic value |
| Composite | address -> city, PIN | Store components needed for queries |
| Single-valued | date of birth | One value per entity |
| Multivalued | phone numbers | Usually separate relation |
| Derived | age from DOB | Prefer source fact; compute when needed |
| Optional | middle name | May be NULL/absent |

“Atomic” depends on intended operations. An address string may be atomic for display but not for city-based querying.

## Strong vs weak entity

- **Strong entity:** has its own identifying key.
- **Weak entity:** cannot be uniquely identified by its attributes alone; identified through owner entity plus partial key.
- Identifying relationship and total participation connect the weak entity to its owner.

Example: `OrderItem` may be identified by `(order_id, line_number)`.

## Relationship degree

- Unary/recursive: employee manages employee.
- Binary: customer places order.
- Ternary: supplier provides part to project.

Do not automatically replace a ternary relationship with three binary relationships; the original joint constraint may be lost.

## Cardinality and participation

| Concept | Question answered |
|---|---|
| Cardinality | Maximum number of related entities? |
| Participation | Must every entity participate? |

- 1:1, 1:N, N:1, M:N describe maxima.
- Total participation means every entity in that set must appear in the relationship.
- Minimum/maximum notation is often clearer than only crow's-foot symbols.

## Mapping ER to relations

### Strong entity

Create a relation containing simple attributes and primary key. Expand composite attributes into required components.

### Multivalued attribute

Create a separate relation containing owner key plus attribute value. The combination commonly forms the key.

### Weak entity

Create a relation with owner primary key as foreign key plus partial key and attributes. Combined owner key + partial key identifies the weak entity.

### 1:N relationship

Place the key of the “1” side as a foreign key on the “N” side. Relationship attributes usually go on the N-side relation.

### 1:1 relationship

Place a foreign key on a side that minimizes NULLs and reflects total participation/ownership; enforce uniqueness. Sometimes merge entities when lifecycles and access patterns justify it.

### M:N relationship

Create a junction relation containing foreign keys to both sides plus relationship attributes. Its key may be the combination or a surrogate plus a unique business constraint.

### N-ary relationship

Create a relation with keys of participating entities and relationship attributes. Determine the key from actual cardinality constraints, not by blindly using all columns.

## Specialization and generalization

- **Generalization:** combine common properties into a superclass.
- **Specialization:** divide a superclass into subclasses.
- **Disjoint:** entity belongs to at most one subclass.
- **Overlapping:** entity may belong to multiple subclasses.
- **Total:** every superclass entity belongs to a subclass.
- **Partial:** some belong to none.

Relational mapping options:

1. One superclass table plus one table per subtype.
2. One table per concrete subtype including inherited fields.
3. One wide table with type discriminator and nullable subtype fields.

Each trades joins, duplication, NULLs and constraint complexity.

## Aggregation

Treats a relationship set as a higher-level entity so it can participate in another relationship. It differs from generalization/inheritance.

## Design workflow

1. Identify business facts and lifecycle boundaries.
2. Identify entities and stable keys.
3. Define relationships, minimum/maximum cardinalities and participation.
4. Record attributes and constraints.
5. Resolve M:N/multivalued structures.
6. Map to relations.
7. Normalize and check dependency semantics.
8. Add physical indexes only after query/access analysis.

## Common modeling mistakes

- Storing comma-separated lists in one column
- Treating changing names/emails as permanent identifiers
- Missing uniqueness beyond surrogate IDs
- Storing derived values without update rules
- Converting every noun into an entity
- Using one giant nullable table for unrelated subtypes
- Placing relationship attributes on an arbitrary entity
- Omitting minimum participation/business constraints

## Common traps

- Cardinality and participation are not synonyms.
- A foreign key does not automatically make an entity weak.
- A junction table may need attributes of its own.
- A surrogate key does not remove the need for a composite uniqueness constraint.
- Derived data can be cached, but then invalidation/consistency must be defined.
- ER design is conceptual; indexes are physical-design choices.

## Interview checks

1. Model students enrolling in courses with grade and semester.
2. Weak entity vs strong entity?
3. How do you map a multivalued attribute?
4. Where should the foreign key go in 1:1 and 1:N relationships?
5. Why does M:N require a junction table?
6. Cardinality vs total participation?
7. Model inheritance in relational tables and explain trade-offs.
8. Why can replacing a ternary relationship with binaries be incorrect?

## 60-second recall

- Entity is an object; relationship is an association; attribute is a property.
- Cardinality gives maximum; participation gives minimum requirement.
- Weak entity uses owner key + partial key.
- 1:N -> FK on N side; M:N -> junction relation.
- Specialization mappings trade joins, duplication and NULLs.

