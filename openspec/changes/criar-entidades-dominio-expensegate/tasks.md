## 1. Enums

- [x] 1.1 Create `academy.jairo.expensegate.enums.Department` with constants `IT`, `ENG` and verify the project compiles (`./mvnw -q compile`)
- [x] 1.2 Create `academy.jairo.expensegate.enums.ExpenseStatus` with constants `SUBMITTED`, `APPROVED`, `REJECTED` and verify the project compiles (`./mvnw -q compile`)

## 2. Independent entities

- [x] 2.1 Create `academy.jairo.expensegate.entity.Authority` (`@Table(name = "tb_authority")`, `id` with `@Id @GeneratedValue(strategy = GenerationType.IDENTITY)`, `name` String, no-args constructor, explicit getters/setters, no Lombok) and verify it compiles (`./mvnw -q compile`)
- [x] 2.2 Create `academy.jairo.expensegate.entity.Role` (`@Table(name = "tb_role")`, `id` IDENTITY, `name` String, `authorities` as the owning `@ManyToMany` side with `@JoinTable(name = "role_authority", ...)` to `Authority`, no-args constructor, explicit getters/setters) and verify it compiles (`./mvnw -q compile`)
- [x] 2.3 Add the inverse side `Authority.roles` (`@ManyToMany(mappedBy = "authorities")` to `Role`) and verify it compiles (`./mvnw -q compile`)

## 3. User and Expense entities

- [x] 3.1 Create `academy.jairo.expensegate.entity.User` (`@Table(name = "tb_user")`, `id` IDENTITY, `username` String, `password` String, `department` `@Enumerated(EnumType.STRING)` `Department`, `roles` as the owning `@ManyToMany` side with `@JoinTable(name = "user_role", ...)` to `Role`, no-args constructor, explicit getters/setters) and verify it compiles (`./mvnw -q compile`)
- [x] 3.2 Add the inverse side `Role.users` (`@ManyToMany(mappedBy = "roles")` to `User`) and verify it compiles (`./mvnw -q compile`)
- [x] 3.3 Create `academy.jairo.expensegate.entity.Expense` (`@Table(name = "tb_expense")`, `id` IDENTITY, `title` String, `amount` `BigDecimal`, `department` `@Enumerated(EnumType.STRING)` `Department`, `owner` as the owning `@ManyToOne` side with `@JoinColumn` to `User`, `status` `@Enumerated(EnumType.STRING)` `ExpenseStatus`, no-args constructor, explicit getters/setters) and verify it compiles (`./mvnw -q compile`)
- [x] 3.4 Add `User.expenses` (`@OneToMany(mappedBy = "owner")` `Set<Expense>`) and verify it compiles (`./mvnw -q compile`)

## 4. Verification

- [x] 4.1 Run `./mvnw -q compile` from the project root and confirm the build succeeds with the four new entities and two new enums, and no other classes (repositories, services, controllers, DTOs, mappers, migrations) were introduced
- [x] 4.2 Manually review the mapping against design.md's Decisions section: `Set`-typed collections, `IDENTITY` generation, `EnumType.STRING`, entity tables named `tb_user`/`tb_expense`/`tb_role`/`tb_authority`, join tables named `user_role` and `role_authority` (unprefixed), no Lombok

## 5. Constraints, constructors, and fetch strategy

- [x] 5.1 On `academy.jairo.expensegate.entity.User`, add `@Column(unique = true, nullable = false)` to `username` and `@Column(nullable = false)` to `password` and `department`, and verify it compiles (`sh mvnw -q compile`)
- [x] 5.2 On `academy.jairo.expensegate.entity.Role` and `academy.jairo.expensegate.entity.Authority`, add `@Column(nullable = false)` to `name`, and verify it compiles (`sh mvnw -q compile`)
- [x] 5.3 On `academy.jairo.expensegate.entity.Expense`, add `@Column(nullable = false)` to `title`, `amount`, `department`, `status`, and `@JoinColumn(nullable = false)` to `owner`, and verify it compiles (`sh mvnw -q compile`)
- [x] 5.4 Add `fetch = FetchType.EAGER` to all four `@ManyToMany` annotations (`User.roles`, `Role.users`, `Role.authorities`, `Authority.roles`) and verify it compiles (`sh mvnw -q compile`)
- [x] 5.5 Add an all-args constructor (scalar attributes only, in field declaration order, excluding `id` and relationship collections) to `User`, `Expense`, `Role`, and `Authority`, alongside the existing no-args constructor, and verify it compiles (`sh mvnw -q compile`)
- [x] 5.6 Run `sh mvnw -q compile` from the project root and confirm the build succeeds; manually re-review `User`, `Expense`, `Role`, `Authority` against the updated design.md Decisions (required/unique columns, `EAGER` fetch on `@ManyToMany`, all-args constructors)

## 6. Join table naming

- [x] 6.1 On `academy.jairo.expensegate.entity.User`, rename the `@JoinTable` for `roles` from `"user_role"` to `"tb_user_role"` and verify it compiles (`sh mvnw -q compile`)
- [x] 6.2 On `academy.jairo.expensegate.entity.Role`, rename the `@JoinTable` for `authorities` from `"role_authority"` to `"tb_role_authority"` and verify it compiles (`sh mvnw -q compile`)
- [x] 6.3 Run `sh mvnw -q compile` from the project root and confirm the build succeeds; manually re-review both `@JoinTable` names against the updated design.md "Table naming" decision

## 7. Align with the reference entity template (`prompts/esperado.md`)

- [x] 7.1 On `User`, `Expense`, `Role`, and `Authority`, remove `strategy = GenerationType.IDENTITY` from `@GeneratedValue` (leave it as plain `@GeneratedValue`) and verify it compiles (`sh mvnw -q compile`)
- [x] 7.2 On `academy.jairo.expensegate.entity.User`, remove the `expenses` field (`@OneToMany(mappedBy = "owner")`) and its `getExpenses`/`setExpenses` methods, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.3 On `academy.jairo.expensegate.entity.Role`, remove the `users` field (`@ManyToMany(mappedBy = "roles")`) and its `getUsers`/`setUsers` methods, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.4 On `academy.jairo.expensegate.entity.Authority`, remove the `roles` field (`@ManyToMany(mappedBy = "authorities")`) and its `getRoles`/`setRoles` methods, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.5 On `academy.jairo.expensegate.entity.Role` and `academy.jairo.expensegate.entity.Authority`, add `unique = true` to the `@Column` on `name` and verify it compiles (`sh mvnw -q compile`)
- [x] 7.6 On `academy.jairo.expensegate.entity.Expense`, replace `@ManyToOne` + `@JoinColumn(name = "owner_id", nullable = false)` on `owner` with `@ManyToOne(fetch = FetchType.LAZY, optional = false)` (no `@JoinColumn`) and verify it compiles (`sh mvnw -q compile`)
- [x] 7.7 Rewrite the all-args constructor of `User` to `User(Long id, String username, String password, Department department, Set<Role> roles)`, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.8 Rewrite the all-args constructor of `Role` to `Role(Long id, String name, Set<Authority> authorities)`, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.9 Rewrite the all-args constructor of `Authority` to `Authority(Long id, String name)`, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.10 Rewrite the all-args constructor of `Expense` to `Expense(Long id, String title, BigDecimal amount, Department department, User owner, ExpenseStatus status)`, and verify it compiles (`sh mvnw -q compile`)
- [x] 7.11 Run `sh mvnw -q compile` from the project root and confirm the build succeeds; manually diff `User`, `Expense`, `Role`, `Authority` field-by-field and constructor-by-constructor against `prompts/esperado.md`
