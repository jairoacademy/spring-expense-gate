Arquivo User

@Entity
@Table(name = "tb_user")
public class User {

@Id
@GeneratedValue
private Long id;

@Column(unique = true, nullable = false)
private String username;

@Column(nullable = false)
private String password;

@Enumerated(EnumType.STRING)
@Column(nullable = false)
private Department department;

@ManyToMany(fetch = FetchType.EAGER)
@JoinTable(name = "tb_user_role",
joinColumns = @JoinColumn(name = "user_id"),
inverseJoinColumns = @JoinColumn(name = "role_id"))
private Set<Role> roles = new HashSet<>();

public User() {
}

public User(Long id, String username, String password, Department department, Set<Role> roles) {
this.id = id;
this.username = username;
this.password = password;
this.department = department;
this.roles = roles;
}
public Long getId() {
return id;
}
public void setId(Long id) {
this.id = id;
}
public String getUsername() {
return username;
}
public void setUsername(String username) {
this.username = username;
}
public String getPassword() {
return password;
}
public void setPassword(String password) {
this.password = password;
}
public Department getDepartment() {
return department;
}
public void setDepartment(Department department) {
this.department = department;
}
public Set<Role> getRoles() {
return roles;
}
public void setRoles(Set<Role> roles) {
this.roles = roles;
}
}
Arquivo Role

@Entity
@Table(name = "tb_role")
public class Role {

@Id
@GeneratedValue
private Long id;

@Column(unique = true, nullable = false)
private String name;

@ManyToMany(fetch = FetchType.EAGER)
@JoinTable(name = "tb_role_authority",
joinColumns = @JoinColumn(name = "role_id"),
inverseJoinColumns = @JoinColumn(name = "authority_id"))
private Set<Authority> authorities = new HashSet<>();

public Role() {
}
public Role(Long id, String name, Set<Authority> authorities) {
this.id = id;
this.name = name;
this.authorities = authorities;
}
public Long getId() {
return id;
}
public void setId(Long id) {
this.id = id;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
public Set<Authority> getAuthorities() {
return authorities;
}
public void setAuthorities(Set<Authority> authorities) {
this.authorities = authorities;
}
}
Arquivo Authority

@Entity
@Table(name = "tb_authority")
public class Authority {
@Id
@GeneratedValue
private Long id;

@Column(unique = true, nullable = false)
private String name;

public Authority() {
}
public Authority(Long id, String name) {
this.id = id;
this.name = name;
}
public Long getId() {
return id;
}
public void setId(Long id) {
this.id = id;
}
public String getName() {
return name;
}
public void setName(String name) {
this.name = name;
}
}
Arquivo Expense

@Entity
@Table(name = "tb_expense")
public class Expense {
@Id
@GeneratedValue
private Long id;
@Column(nullable = false)
private String title;
@Column(nullable = false)
private BigDecimal amount;
@Enumerated(EnumType.STRING)
@Column(nullable = false)
private Department department;
@ManyToOne(fetch = FetchType.LAZY, optional = false)
private User owner;
@Enumerated(EnumType.STRING)
@Column(nullable = false)
private ExpenseStatus status;
public Expense() {
}
public Expense(Long id, String title, BigDecimal amount, Department department, User owner, ExpenseStatus status) {
this.id = id;
this.title = title;
this.amount = amount;
this.department = department;
this.owner = owner;
this.status = status;
}
public Long getId() {
return id;
}
public void setId(Long id) {
this.id = id;
}
public String getTitle() {
return title;
}
public void setTitle(String title) {
this.title = title;
}
public BigDecimal getAmount() {
return amount;
}
public void setAmount(BigDecimal amount) {
this.amount = amount;
}
public Department getDepartment() {
return department;
}
public void setDepartment(Department department) {
this.department = department;
}
public User getOwner() {
return owner;
}
public void setOwner(User owner) {
this.owner = owner;
}
public ExpenseStatus getStatus() {
return status;
}
public void setStatus(ExpenseStatus status) {
this.status = status;
}
}