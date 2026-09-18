## Purpose

Define the core domain entities and classifications of the ExpenseGate application — users, roles, authorities, and expenses — and the relationships that hold between them, independent of how they are persisted or exposed.

## ADDED Requirements

### Requirement: User domain entity
The system SHALL represent a user of ExpenseGate as a distinct domain entity identified uniquely, with a username, a password, and an assigned department. The username SHALL be unique among all users. Password and department are required and SHALL NOT be absent. A user MAY submit zero or more expenses and MAY hold zero or more roles.

#### Scenario: User holds roles
- **WHEN** one or more roles are assigned to a user
- **THEN** those roles are reachable as that user's roles

#### Scenario: Username uniqueness is enforced
- **WHEN** a new user is recorded with a username that already belongs to an existing user
- **THEN** the system rejects the new user

#### Scenario: Password and department are required
- **WHEN** a user is recorded without a password or without a department
- **THEN** the system rejects the user

### Requirement: Expense domain entity
The system SHALL represent an expense as a distinct domain entity identified uniquely, with a title, a monetary amount, a department, an owning user, and a status. Every expense SHALL be associated with exactly one owning user, and its title, amount, department, owner, and status are all required and SHALL NOT be absent.

#### Scenario: Expense is linked to its owner
- **WHEN** an expense is recorded
- **THEN** it references exactly one user as its owner

#### Scenario: Expense carries a status
- **WHEN** an expense is recorded
- **THEN** it has one status drawn from the defined expense status classification

#### Scenario: Expense fields are all required
- **WHEN** an expense is recorded missing its title, amount, department, owner, or status
- **THEN** the system rejects the expense

### Requirement: Role domain entity
The system SHALL represent a role as a distinct domain entity identified uniquely, with a name. The name is required, SHALL NOT be absent, and SHALL be unique among all roles. A role MAY be held by zero or more users and MAY be granted zero or more authorities.

#### Scenario: Role is shared by multiple users
- **WHEN** the same role is assigned to more than one user
- **THEN** each of those users has that role among their roles

#### Scenario: Role grants authorities
- **WHEN** one or more authorities are associated with a role
- **THEN** those authorities are reachable as that role's authorities

#### Scenario: Role name is required
- **WHEN** a role is recorded without a name
- **THEN** the system rejects the role

#### Scenario: Role name uniqueness is enforced
- **WHEN** a new role is recorded with a name that already belongs to an existing role
- **THEN** the system rejects the new role

### Requirement: Authority domain entity
The system SHALL represent an authority (permission) as a distinct domain entity identified uniquely, with a name. The name is required, SHALL NOT be absent, and SHALL be unique among all authorities. An authority MAY be granted to zero or more roles.

#### Scenario: Authority is shared by multiple roles
- **WHEN** the same authority is associated with more than one role
- **THEN** each of those roles has that authority among its authorities

#### Scenario: Authority name is required
- **WHEN** an authority is recorded without a name
- **THEN** the system rejects the authority

#### Scenario: Authority name uniqueness is enforced
- **WHEN** a new authority is recorded with a name that already belongs to an existing authority
- **THEN** the system rejects the new authority

### Requirement: Department classification
The system SHALL classify both users and expenses by department, restricted to a fixed, closed set of values: `IT` and `ENG`. No other department value is valid.

#### Scenario: Only defined departments are accepted
- **WHEN** a department value is assigned to a user or an expense
- **THEN** the value is one of `IT` or `ENG`

### Requirement: Expense status classification
The system SHALL classify an expense's approval state using a fixed, closed set of values: `SUBMITTED`, `APPROVED`, `REJECTED`. No other status value is valid.

#### Scenario: Only defined statuses are accepted
- **WHEN** a status value is assigned to an expense
- **THEN** the value is one of `SUBMITTED`, `APPROVED`, or `REJECTED`
