## Why

O ExpenseGate ainda não possui nenhuma classe de domínio: o projeto contém apenas o esqueleto gerado pelo Spring Initializr, embora `spring-boot-starter-data-jpa`, `spring-boot-starter-security` e o driver PostgreSQL já estejam no `pom.xml`. É preciso modelar as entidades JPA e os enums que sustentam o domínio de gestão de despesas (usuários, papéis/permissões e despesas) antes que qualquer repository, service ou controller possa ser construído.

## What Changes

- Criar a entidade `User` (id, username, password, department, expenses 1:N mapeada por `owner`, roles N:N).
- Criar a entidade `Expense` (id, title, amount, department, owner N:1 com `User`, status).
- Criar a entidade `Role` (id, name, users N:N — lado inverso de `User.roles`, authorities N:N).
- Criar a entidade `Authority` (id, name, roles N:N — lado inverso de `Role.authorities`).
- Criar o enum `Department` (`IT`, `ENG`).
- Criar o enum `ExpenseStatus` (`SUBMITTED`, `APPROVED`, `REJECTED`).
- Fixar as convenções técnicas do projeto para essas classes: sem Lombok (getters/setters explícitos), `@GeneratedValue(strategy = GenerationType.IDENTITY)`, `@Enumerated(EnumType.STRING)`, tabelas de junção `user_role` e `role_authority`.

Fora do escopo desta mudança (proposto em mudanças futuras separadas): repositories, services, controllers, DTOs, mappers, configuração de segurança, endpoints REST, migrations (Flyway/Liquibase) e testes automatizados.

## Capabilities

### New Capabilities
- `domain-model`: entidades JPA e enums que formam o modelo de domínio do ExpenseGate (usuários, papéis, permissões e despesas), incluindo seus relacionamentos e convenções de mapeamento.

### Modified Capabilities
(nenhuma — projeto greenfield, não há specs existentes)

## Impact

- **Código afetado**: novo pacote `academy.jairo.expensegate.entity` (classes `User`, `Expense`, `Role`, `Authority`) e novo pacote `academy.jairo.expensegate.enums` (enums `Department`, `ExpenseStatus`).
- **Banco de dados**: tabelas para as quatro entidades e duas tabelas de junção (`user_role`, `role_authority`); nenhuma migration é criada nesta mudança — o schema dependerá da estratégia de DDL configurada (ex.: `ddl-auto` do Hibernate) até que migrations sejam propostas separadamente.
- **Dependências**: nenhuma dependência nova; usa `spring-boot-starter-data-jpa` já presente no `pom.xml`.
- **Sistemas**: nenhum impacto em APIs REST, segurança ou testes automatizados — esses ficam fora do escopo.