# Prompt: criar entidades de domínio (ExpenseGate)

Comando pronto para colar no Claude Code para iniciar o fluxo OpenSpec
(gera apenas artefatos de planejamento — proposal/spec/design/tasks —,
não edita código; a implementação vem depois via `/opsx:apply`).

```
/opsx:propose criar-entidades-dominio-expensegate

Criar apenas as entidades JPA (User, Expense, Role, Authority) e os enums
(Department, ExpenseStatus) do domínio ExpenseGate, conforme especificação
abaixo. Escopo desta mudança: SOMENTE essas classes de entidade e enum.

Entidades:
- User: id, username (String), password (String), department (Department),
  expenses (1:N com Expense, mappedBy owner), roles (N:N com Role)
- Expense: id, title (String), amount (BigDecimal), department (Department),
  owner (N:1 com User), status (ExpenseStatus)
- Role: id, name (String), users (N:N com User, lado inverso),
  authorities (N:N com Authority)
- Authority: id, name (String), roles (N:N com Role, lado inverso)

Enums:
- Department: IT, ENG
- ExpenseStatus: SUBMITTED, APPROVED, REJECTED

Convenções técnicas que as specs/tasks devem fixar:
- Sem Lombok: getters/setters explícitos
- @GeneratedValue(strategy = GenerationType.IDENTITY)
- @Enumerated(EnumType.STRING)
- Tabelas de junção: user_role, role_authority

Fora do escopo (não incluir nas tasks): repositories, services, controllers,
DTOs, mappers, configuração de segurança, endpoints REST, migrations
(Flyway/Liquibase) e testes automatizados. Isso será proposto em mudanças
futuras separadas.
```

## Depois de rodar

Revise o `tasks.md` gerado antes de rodar `/opsx:apply` — se alguma task
extrapolar o escopo (ex.: sugerir criar repository "de brinde"), corte ali,
antes da implementação.
