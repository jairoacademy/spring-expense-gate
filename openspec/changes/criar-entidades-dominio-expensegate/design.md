## Context

See proposal.md - Why. The project has no domain classes yet, only `spring-boot-starter-data-jpa`, `spring-boot-starter-security`, and the PostgreSQL driver on the classpath, with layered packages already established by project convention (`entity`, `enums`, `repository`, `service`, `controller`).

This design covers the JPA mapping decisions needed to introduce `User`, `Expense`, `Role`, `Authority`, `Department`, and `ExpenseStatus` under `academy.jairo.expensegate.entity` and `academy.jairo.expensegate.enums`.

## Goals / Non-Goals

**Goals:**
- Fix the concrete JPA annotations and mapping style for each entity and relationship described in `specs/domain-model/spec.md`.
- Fix identifier generation, enum persistence, and join table naming so later changes (repositories, services, migrations) build on a stable, agreed shape.

**Non-Goals:**
- No repositories, services, controllers, DTOs, mappers, or security configuration (see proposal.md - Impact).
- No Flyway/Liquibase migrations - schema generation strategy (e.g., Hibernate `ddl-auto`) is left to existing/local configuration and is not changed here.
- No Bean Validation (`jakarta.validation`) annotations such as `@NotNull`/`@Size` - required and uniqueness constraints are expressed at the JPA/column level (`@Column(nullable = false, unique = ...)`, `@JoinColumn(nullable = false)`) instead; see Decisions.

## Decisions

**No Lombok - explicit getters/setters**
Project convention for this change (per prompt) is to avoid Lombok. Each entity gets hand-written getters/setters, a no-args constructor (required by JPA), and `equals`/`hashCode` are left to JPA defaults (identity-based) rather than field-based, since these are mutable entities with database-generated identifiers - field-based equals/hashCode on mutable entities is a common source of bugs with Hibernate proxies and Set membership before persist.

**Identifier generation: `@GeneratedValue` with no explicit strategy (JPA default `AUTO`)**
Reverts the earlier explicit `GenerationType.IDENTITY` choice, per the project's reference entity template (`prompts/esperado.md`). `AUTO` lets the JPA provider (Hibernate) pick the generation strategy for the target database; no strategy is pinned unless a future requirement needs one.

**Enum persistence: `@Enumerated(EnumType.STRING)`**
Stores enum names as text (`IT`, `ENG`, `SUBMITTED`, ...) rather than ordinal integers, so the database stays readable and safe to reorder enum constants later. Matches the explicit convention in the prompt.

**Relationship mapping: unidirectional by default**
Only the owning/natural side of each relationship is mapped as a Java field; the inverse (`mappedBy`) side is omitted unless a specific requirement needs direct navigation from that side. None of the current spec scenarios require navigating from the "many" or "granted-to" side, so:
- `Expense.owner` (N:1, owning side, default `@JoinColumn`, required): the only mapped side of the `User`/`Expense` relationship. `User` does **not** expose an `expenses` collection.
- `User.roles` (N:N, owning side, `@JoinTable(name = "tb_user_role", ...)`): the only mapped side of the `User`/`Role` relationship. `Role` does **not** expose a `users` collection.
- `Role.authorities` (N:N, owning side, `@JoinTable(name = "tb_role_authority", ...)`): the only mapped side of the `Role`/`Authority` relationship. `Authority` does **not** expose a `roles` collection.
- Collection type: `Set` (not `List`) for all many-to-many collections, since none of these relationships have a meaningful order and `Set` avoids duplicate membership by construction.
- Fetch type: `@ManyToMany` relationships (`User.roles`, `Role.authorities`) use `fetch = FetchType.EAGER` explicitly. `@ManyToOne` (`Expense.owner`) uses `fetch = FetchType.LAZY` explicitly, combined with `optional = false` (see "Required and unique columns" below).

**Table naming: `tb_<name>` prefix for every table**
All six tables get an explicit name with a `tb_` prefix: the four main entities via `@Table(name = ...)` (`tb_user`, `tb_expense`, `tb_role`, `tb_authority`) and the two join tables via `@JoinTable(name = ...)` on the owning side of each many-to-many (`tb_user_role`, `tb_role_authority`). This supersedes the earlier decision to leave join tables unprefixed (`user_role`, `role_authority`) - consistency across every table in the schema was preferred over distinguishing entity vs. join tables by name.

**Amount type**
`Expense.amount` uses `java.math.BigDecimal`, per the prompt, mapped with JPA's default numeric mapping (`@Column` without an explicit precision/scale, since none was specified) - fine for a domain model with no persistence-layer decisions made yet; precision/scale can be tightened when migrations are proposed.

**Required and unique columns via `@Column`/`@ManyToOne(optional = ...)`**
Per the updated spec requirements: `User.username`, `Role.name`, and `Authority.name` all get `@Column(unique = true, nullable = false)` - each must exist to be meaningfully unique, so `nullable = false` is bundled with the uniqueness request. `User.password` and `User.department` get `@Column(nullable = false)`. Every `Expense` attribute is required: `title`, `amount`, `department`, `status` via `@Column(nullable = false)`, and `owner` via `@ManyToOne(optional = false)` - no explicit `@JoinColumn` is used, so the join column keeps JPA's default name (`owner_id`, derived from the field name and the referenced primary key) and `optional = false` is what marks the underlying foreign key column `NOT NULL`, matching the project's reference entity template. No database-level defaults or application-level validation messages are introduced - violations surface as the underlying JPA/database constraint-violation exception.

**All-args constructor alongside the no-args constructor**
Each entity keeps its required no-args constructor (JPA) and gains a second constructor taking every mapped attribute in field declaration order, **including `id`** and **including the owning-side relationship field** - `User(id, username, password, department, roles)`, `Expense(id, title, amount, department, owner, status)`, `Role(id, name, authorities)`, `Authority(id, name)`. This reverses the earlier decision (which excluded `id` and relationship fields), aligning with the project's reference entity template (`prompts/esperado.md`). Since relationships are unidirectional by default (see "Relationship mapping"), there are no inverse/`mappedBy` fields to exclude.

## Risks / Trade-offs

- [`@ManyToMany` forced to `EAGER`] → Loading a `User` or `Role` now eagerly loads its full `roles`/`authorities` graph, which can be expensive as data grows and risks N+1-style over-fetching well before a service layer exists to control it. Mitigation: explicit requirement for now; revisit once repositories/services define real access patterns.
- [Unidirectional relationships only] → Without `User.expenses`, `Role.users`, or `Authority.roles`, code cannot navigate from the "one"/"granted-to" side directly; reaching e.g. "all expenses for a user" requires a query (future repository), not object navigation. Mitigation: accepted per the project's reference entity template; add the inverse side later, scoped to a specific requirement, if direct navigation is ever needed.
- [`GenerationType.AUTO` instead of `IDENTITY`] → Hibernate's resolution of `AUTO` for PostgreSQL may pick a sequence-based generator (e.g. a shared or per-entity sequence) rather than a native `SERIAL`/`IDENTITY` column, which can behave differently under concurrent inserts and in generated DDL than the previously chosen `IDENTITY` strategy. Mitigation: confirm the actual generator Hibernate selects for this Hibernate/PostgreSQL combination once migrations are introduced; pin a strategy explicitly if specific behavior is required.
- [No explicit `@Column` length/precision] → Relies on JPA/Hibernate and PostgreSQL defaults, which may need tightening once migrations are introduced. Mitigation: revisit alongside the Flyway/Liquibase migration change.
- [`Set`-based collections with Hibernate proxies] → `equals`/`hashCode` left at Object identity avoids the classic Hibernate `Set`+field-equals pitfall, but means two transient (unsaved) instances are never "equal" even with matching data. Accepted trade-off: no domain logic in this change relies on transient-instance equality.
