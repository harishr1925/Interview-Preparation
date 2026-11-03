# Spring Data JPA & Hibernate Interview Questions & Answers

> Part A (Q1–20) is from your notes. Part B (Q21 onwards) is extra questions commonly asked in interviews.

# PART A: From Your Notes

## 🔹 Model (Entity) Layer

**1. What is an Entity / the Model layer in JPA?**
👉 Create a simple POJO (Plain Old Java Class) and add JPA annotations to it. The class is then mapped to a database table.

```java
@Entity
@Table(name = "student")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "student_name")
    private String name;

    private String email;
}
```

**2. What does `@Entity` do?**
👉 It defines that a class can be mapped to a table.

**3. What does `@Id` do?**
👉 It specifies the primary key of the entity.

**4. What does `@GeneratedValue` do?**
👉 It specifies the primary key generation strategy. It instructs the database to generate a value for this field automatically. If the strategy is not specified, **AUTO** is used by default.

**5. What are the other commonly used JPA annotations?**

| Annotation | Purpose |
|---|---|
| `@Table(name = "table_name")` | Specifies the table name (optional if same as class name) |
| `@GeneratedValue(strategy = ...)` | Auto-generates primary key values (AUTO, IDENTITY, etc.) |
| `@Column(name = "column_name")` | Maps a field to a specific column (optional if name matches) |
| `@Transient` | Marks a field that should **not** be persisted in the database |
| `@Enumerated(EnumType.STRING)` | Stores an enum as a string in the DB |

---

## 🔹 Spring Data JPA

**6. What is Spring Data JPA?**
👉 Spring Data JPA is a part of the Spring Data project that provides a high-level abstraction for the Java Persistence API (JPA). It simplifies CRUD operations, query execution, pagination, and dynamic query generation in a Spring application using standard repository interfaces. It is typically used with **Hibernate** as the default JPA provider.

**7. What are the key features of Spring Data JPA?**
👉
- **Simplified CRUD:** no boilerplate DAO code, just extend `JpaRepository` or `CrudRepository`
- **Automatic query generation:** from method names
- **Custom JPQL / native SQL:** using `@Query`
- **Pagination and sorting:** built-in `Pageable` and `Sort`
- **Transaction management:** works with Spring's `@Transactional`

**8. What is automatic query generation (derived queries)?**
👉 You define a method by following a naming convention, and Spring generates the query.

```java
List<User> findByLastName(String lastName);
```

**9. How do you write custom queries?**
👉 Using `@Query` with JPQL or native SQL.

```java
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);
```

**10. What are the key components of Spring Data JPA?**

| Component | Purpose |
|---|---|
| `@Entity` | Maps a Java class to a database table |
| `@Id` | Marks the primary key |
| `JpaRepository` | Interface that provides built-in CRUD, pagination, and query methods |
| `@Query` | Defines custom JPQL or SQL queries |

---

## 🔹 Hibernate

**11. What is Hibernate?**
👉 Hibernate is an **Object-Relational Mapping (ORM)** framework for Java. It simplifies the development of Java applications that interact with relational databases by mapping Java objects to database tables.

**12. What does Hibernate do?**
👉 It acts as a **bridge** between the Java application and the database:
- Converts Java objects into database rows (and vice versa)
- Manages database connections
- Executes SQL automatically behind the scenes

**13. What are the key features of Hibernate?**
👉
- **ORM:** maps Java classes to tables using annotations or XML
- **HQL:** object-oriented query language similar to SQL
- **Automatic table creation:** generates tables from your Java classes
- **Transaction management:** integrates with JTA and JDBC (commit/rollback)
- **Caching:** first-level (default) and second-level
- **Lazy and eager loading**
- **Dirty checking**
- **Database independence:** MySQL, Oracle, PostgreSQL with minimal configuration changes
- **Annotation and XML configuration**
- **Connection pooling support**

**14. What is HQL?**
👉 Hibernate Query Language: an object-oriented query language similar to SQL, but it works with **entities and their properties**, not tables and columns.

```java
"FROM Student s WHERE s.name = :name"
```

**15. What is Hibernate caching?**
👉 Supports first-level (default, per session) and second-level (shared across sessions) caching for faster data access.

**16. What is lazy and eager loading?**
👉
- **Lazy:** data is loaded only when it is accessed
- **Eager:** data is loaded immediately with the parent entity

**In detail:** When an entity has a relationship (for example, a `Department` has many `Employee`s), the fetch type decides **when** the related data is loaded from the database.

**Lazy loading** (`FetchType.LAZY`)
- Hibernate loads only the parent. The related data is **not** fetched yet.
- Instead of the real data, Hibernate puts a **proxy** (placeholder) in the field.
- The extra SELECT runs only when you first access it, for example `department.getEmployees().size()`.
- It saves memory and avoids unnecessary queries when you don't need the related data.

**Eager loading** (`FetchType.EAGER`)
- Hibernate loads the parent **and** the related data together, using a JOIN or an immediate second SELECT.
- The data is always available, even after the session is closed.
- It costs more: you load related data even when you don't use it, and it can slow down queries badly if many relationships are eager.

```java
@Entity
public class Department {
    @Id @GeneratedValue
    private Long id;
    private String name;

    // LAZY: employees are loaded only when getEmployees() is accessed
    @OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
    private List<Employee> employees;
}

@Entity
public class Employee {
    @Id @GeneratedValue
    private Long id;

    // EAGER: department is loaded immediately with the employee
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "dept_id")
    private Department department;
}
```

**What happens in SQL:**
```java
Department d = repo.findById(1L).get();   // SELECT * FROM department WHERE id = 1
d.getName();                              // no query, already loaded
d.getEmployees().size();                  // NOW: SELECT * FROM employee WHERE dept_id = 1   (lazy trigger)
```
With EAGER on `employees`, both queries (or a JOIN) would run at the first line.

| Lazy | Eager |
|---|---|
| Loaded **on demand** | Loaded **immediately** |
| Fewer queries up front, less memory | Related data always ready |
| Can cause `LazyInitializationException` if accessed after the session closes | No lazy exception |
| Can cause the N+1 problem in loops | Can load too much data (over-fetching) |
| Recommended default | Use only when the data is always needed |

**Default fetch types:**
- `@OneToMany`, `@ManyToMany` → **LAZY**
- `@ManyToOne`, `@OneToOne` → **EAGER**

**Common problems:**
- **LazyInitializationException:** accessing a lazy field after the session/transaction is closed (see Q46).
- **N+1 problem:** looping over parents and touching a lazy child collection runs one query per parent (see Q45).

**Best practice:** Keep associations LAZY, and when you need related data, fetch it explicitly for that query using `JOIN FETCH` or `@EntityGraph`.

🎯 **Interview Line:** "Lazy loads related data only when it is accessed, eager loads it along with the parent. I keep relationships lazy by default and fetch what I need per query using JOIN FETCH or EntityGraph."

**17. What is dirty checking?**
👉 Hibernate automatically detects changes in managed objects and updates the database accordingly, **without manual update statements**.

```java
@Transactional
public void rename(Long id, String name) {
    Student s = repo.findById(id).get();
    s.setName(name);   // no save() needed, the UPDATE runs at commit
}
```

**In detail: when does the UPDATE run if I call multiple setters?**
👉 The UPDATE runs **only once**, not at each setter line. The setters only change the Java object in memory, and Hibernate does not talk to the database at that point.

When the transaction is about to commit (or at an earlier flush), Hibernate compares each managed entity with the snapshot taken when it was loaded. If anything changed, it generates **one UPDATE** for that entity, covering all the changed fields.

```java
@Transactional
public void updateStudent(Long id, String name, int age) {
    Student s = repo.findById(id).get();  // SELECT, snapshot taken
    s.setName(name);                      // memory only, no SQL
    s.setAge(age);                        // memory only, no SQL
}                                         // commit → flush → ONE UPDATE
```

```sql
UPDATE student SET name=?, age=?, email=? WHERE id=?
```

**Two details worth knowing for interviews:**
- **All columns are included by default.** Hibernate lists every column in the UPDATE, even the ones you didn't change. This lets it reuse the same prepared statement. If you want only the changed columns, add `@DynamicUpdate` on the entity.
- **If you set the same value, no UPDATE runs.** If `setName("Harish")` is called when the name is already "Harish", the snapshot matches and nothing is sent.

```java
@Entity
@DynamicUpdate   // UPDATE student SET name=?, age=? WHERE id=?  (only changed columns)
public class Student { ... }
```

**Can the UPDATE run earlier than commit?**
👉 Yes. Before a query runs inside the same transaction (like a `findByXxx()` or a JPQL query), Hibernate may flush pending changes first so the query sees them. This is the default flush mode, `AUTO`. You can also force it with `repo.flush()`. A flush runs the SQL but does **not** commit, so it can still be rolled back.

🎯 **Interview Line:** "Setters only modify the managed object in memory. Hibernate runs a single UPDATE at flush time, usually at commit, containing all the changed fields."

**18. How does Hibernate automatically create tables?**
👉 Through the `ddl-auto` setting:

```properties
spring.jpa.hibernate.ddl-auto=update
```

**19. Why use Hibernate?**
👉
- Reduces boilerplate JDBC code
- Better maintainability and portability
- Improves productivity with automated CRUD operations

**20. How does Hibernate achieve database independence?**
👉 It generates SQL for the target database through its **dialect** (MySQL, PostgreSQL, Oracle...). Change the DB and the driver/dialect, and the Java code stays the same.

---

# PART B: Extra Questions Expected in Interviews

## 🔹 JPA, Hibernate & Spring Data JPA: Basics

**21. What is the difference between JPA, Hibernate and Spring Data JPA?**

| Name | What it is |
|---|---|
| **JPA** | A **specification** (`jakarta.persistence`), a set of interfaces and rules |
| **Hibernate** | An **implementation** of JPA (others: EclipseLink, OpenJPA) |
| **Spring Data JPA** | A Spring layer **on top of JPA** that removes repository boilerplate |

Flow: `Spring Data JPA → JPA → Hibernate → JDBC → Database`

**22. What is ORM and why do we need it?**
👉 Object-Relational Mapping maps objects to tables, so you work with Java objects instead of SQL and ResultSets. It removes boilerplate JDBC and mapping code.

**23. What are the requirements for an entity class?**
👉
- Annotated with `@Entity`
- Has an `@Id` field
- Has a no-arg constructor (public or protected)
- Class should not be `final` (Hibernate needs to create proxies for lazy loading)

**24. What is the difference between JPQL, HQL and native SQL?**

| Type | Description |
|---|---|
| JPQL | Standard JPA query language, works on entities |
| HQL | Hibernate's query language (superset of JPQL) |
| Native SQL | Plain database SQL, DB-specific |

**25. What is the JPA EntityManager?**
👉 The core JPA interface used to persist, find, update and delete entities and to run queries. It manages the **persistence context**. (In Hibernate, `Session` is the equivalent; `EntityManager` wraps a `Session`.)

**26. What is the Persistence Context?**
👉 A set of managed entity instances for the current unit of work. It is the **first-level cache**, and it is the place where dirty checking happens.

**27. What are the entity lifecycle states?**
👉
- **Transient / New:** created with `new`, not associated with a session, no DB row
- **Managed / Persistent:** attached to the persistence context, changes are tracked
- **Detached:** was managed, but the session is closed or the entity was detached
- **Removed:** scheduled for deletion

---

## 🔹 ID Generation & Mapping Annotations

**28. What are the `@GeneratedValue` strategies?**

| Strategy | Meaning |
|---|---|
| `AUTO` | The provider picks based on the DB (default) |
| `IDENTITY` | DB auto-increment column (MySQL, SQL Server) |
| `SEQUENCE` | DB sequence (PostgreSQL, Oracle) |
| `TABLE` | A separate table simulates a sequence (rarely used, slow) |

**29. Why does IDENTITY disable JDBC batch inserts in Hibernate?**
👉 With IDENTITY, Hibernate needs the generated ID right after each insert, so it must run each INSERT immediately. SEQUENCE lets Hibernate fetch IDs in advance, so batching works.

**30. What is the difference between `@Column` attributes?**
👉 `name`, `nullable`, `unique`, `length`, `precision/scale`, `insertable`, `updatable`.

```java
@Column(name = "email", nullable = false, unique = true, length = 100)
private String email;
```

**31. What is the difference between `EnumType.ORDINAL` and `EnumType.STRING`?**
👉 `ORDINAL` (the default) stores the enum position (0, 1, 2). Reordering or inserting an enum value breaks existing data. `STRING` stores the name, so it is safer and recommended.

**32. What are `@Embedded` and `@Embeddable`?**
👉 `@Embeddable` marks a class whose fields are stored as columns of the owning entity's table. `@Embedded` uses it in the entity (for example, an `Address` inside `Customer`).

**33. What is `@MappedSuperclass`?**
👉 A base class whose fields are inherited by child entities but which is **not itself an entity or table**. It is used for common fields like `id`, `createdAt`, `updatedAt`.

**34. What are the inheritance mapping strategies?**

| Strategy | Description |
|---|---|
| `SINGLE_TABLE` (default) | One table for the whole hierarchy, with a discriminator column |
| `JOINED` | One table per class, joined on the primary key |
| `TABLE_PER_CLASS` | One full table per concrete class |

**35. How do you add auditing (createdAt / updatedAt)?**
```java
@EnableJpaAuditing   // on a config class

@Entity
@EntityListeners(AuditingEntityListener.class)
public class Student {
    @CreatedDate
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

**36. What is `@Version`?**
👉 It enables **optimistic locking**. Hibernate adds `WHERE version = ?` to updates. If another transaction changed the row, the update fails with `OptimisticLockException`.

---

## 🔹 Relationships

**37. What are the relationship annotations in JPA?**
👉 `@OneToOne`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`.

**38. Give an example of a bidirectional One-to-Many.**
```java
@Entity
public class Department {
    @Id @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Employee> employees = new ArrayList<>();
}

@Entity
public class Employee {
    @Id @GeneratedValue
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "dept_id")
    private Department department;
}
```

**39. What is `mappedBy`? What is the owning side?**
👉 `mappedBy` marks the **inverse (non-owning) side**. The **owning side** is the one with the foreign key (`@JoinColumn`), and it controls the DB column. `mappedBy` refers to the field name on the owning side.

**40. What are `@JoinColumn` and `@JoinTable`?**
👉 `@JoinColumn` defines the foreign key column. `@JoinTable` defines the join (link) table, mostly used for `@ManyToMany`.

**41. Show a Many-to-Many example.**
```java
@ManyToMany
@JoinTable(name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id"))
private Set<Course> courses = new HashSet<>();
```

**42. What are the cascade types?**
👉 `PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`, `ALL`. They propagate an operation from the parent to its children.

**43. What is `orphanRemoval`?**
👉 When a child is removed from the parent's collection, it is **deleted from the database** too. (`CascadeType.REMOVE` only deletes children when the parent is deleted.)

**44. What are the default fetch types?**

| Relationship | Default |
|---|---|
| `@OneToOne` | EAGER |
| `@ManyToOne` | EAGER |
| `@OneToMany` | LAZY |
| `@ManyToMany` | LAZY |

Best practice: make everything LAZY and fetch what you need per query.

**45. What is the N+1 problem? How do you solve it?**
👉 One query loads N parents, then **N more queries** load each parent's lazy children.

Solutions:
- `JOIN FETCH` in JPQL
- `@EntityGraph`
- `@BatchSize` / `hibernate.default_batch_fetch_size`
- DTO projection

```java
@Query("SELECT d FROM Department d JOIN FETCH d.employees")
List<Department> findAllWithEmployees();

@EntityGraph(attributePaths = "employees")
List<Department> findAll();
```

**46. What is LazyInitializationException and how do you fix it?**
👉 It is thrown when you access a lazy association **after the session is closed**.

Fixes:
- Fetch the data inside the transaction (`JOIN FETCH` / `@EntityGraph`)
- Use a DTO projection
- Keep the access inside `@Transactional`

Avoid switching everything to EAGER.

**47. Why do bidirectional relationships cause infinite recursion in JSON responses?**
👉 Parent → child → parent → ... when Jackson serializes it (the same happens in `toString()`). Fix: return **DTOs**, or use `@JsonIgnore` / `@JsonManagedReference` + `@JsonBackReference`. Don't use Lombok `@Data` on entities with relations.

---

## 🔹 Spring Data JPA Repositories

**48. What is the difference between CrudRepository, PagingAndSortingRepository and JpaRepository?**

| Interface | Adds |
|---|---|
| `CrudRepository` | Basic CRUD |
| `PagingAndSortingRepository` | Pagination and sorting |
| `JpaRepository` | Everything above + `flush()`, `saveAllAndFlush()`, batch delete, JPA-specific features |

**49. What are the common repository methods?**
👉 `save()`, `saveAll()`, `findById()`, `findAll()`, `existsById()`, `count()`, `deleteById()`, `delete()`, `deleteAll()`, `flush()`.

**50. How does `save()` work: insert or update?**
👉 If the entity is **new** (id is null, or the `@Version` is null), it calls `persist()` (INSERT). Otherwise it calls `merge()` (SELECT + UPDATE).

**51. What is the difference between `findById()` and `getReferenceById()`?**

| `findById()` | `getReferenceById()` |
|---|---|
| Hits the DB immediately | Returns a **lazy proxy** with no DB hit |
| Returns `Optional` | Returns the proxy (throws `EntityNotFoundException` on access if missing) |

**52. What are the common derived-query keywords?**
👉 `And`, `Or`, `Between`, `LessThan`, `GreaterThan`, `Like`, `Containing`, `StartingWith`, `In`, `IsNull`, `OrderBy`, `Top`/`First`, `Distinct`, `existsBy`, `countBy`, `deleteBy`.

```java
List<Student> findByNameContainingIgnoreCaseOrderByIdDesc(String name);
boolean existsByEmail(String email);
List<Student> findTop3ByOrderByMarksDesc();
```

**53. How do you write a native query?**
```java
@Query(value = "SELECT * FROM student WHERE email = ?1", nativeQuery = true)
Student findByEmailNative(String email);
```

**54. How do you run an UPDATE or DELETE with `@Query`?**
👉 Add `@Modifying` and a transaction.

```java
@Modifying
@Transactional
@Query("UPDATE Student s SET s.email = :email WHERE s.id = :id")
int updateEmail(@Param("id") Long id, @Param("email") String email);
```
Use `@Modifying(clearAutomatically = true)` so the persistence context doesn't keep stale entities.

**55. How do you implement pagination and sorting?**
```java
Pageable pageable = PageRequest.of(0, 10, Sort.by("name").descending());
Page<Student> page = repo.findAll(pageable);

page.getContent();        // the list
page.getTotalElements();  // total records
page.getTotalPages();
```

**56. What is the difference between Page, Slice and List?**
👉
- `Page` → content + total count (runs an extra COUNT query)
- `Slice` → content + "has next?" (no count query, cheaper)
- `List` → just the data

**57. What is a projection?**
👉 Fetching only selected columns instead of the whole entity, using an **interface** or a **DTO/record**.

```java
interface StudentView {
    String getName();
    String getEmail();
}
List<StudentView> findByNameStartingWith(String prefix);
```

**58. What are `@NamedQuery` and Specifications?**
👉 `@NamedQuery` defines a reusable JPQL query on the entity. **Specifications** (`JpaSpecificationExecutor`) build **dynamic, type-safe filters** using the Criteria API (for example, search screens with optional filters).

**59. How do you add a custom repository method with your own implementation?**
👉 Create a custom interface and an implementation class named `<Interface>Impl`, then extend the custom interface in your repository.

**60. Which is better: `deleteById()` or a `@Modifying` delete query?**
👉 `deleteById()` loads the entity first (SELECT, then DELETE), and cascades work. A `@Modifying` bulk query runs a single DELETE (faster), but it bypasses the persistence context and cascades/lifecycle callbacks.

---

## 🔹 Hibernate Internals

**61. What are SessionFactory and Session?**
👉
- `SessionFactory` → heavyweight, thread-safe, **one per application**, created at startup
- `Session` → lightweight, **not thread-safe**, one per unit of work, holds the first-level cache

**62. What is the difference between first-level and second-level cache?**

| First-level | Second-level |
|---|---|
| Per Session / persistence context | Per SessionFactory (shared) |
| Enabled by default | Optional, needs a provider (Ehcache, Infinispan) |
| Cleared when the session closes | Lives across sessions |

There is also a **query cache** for query results.

**63. What is the difference between `get()` and `load()`? (Hibernate Session)**
👉 `get()` hits the DB immediately and returns `null` if not found. `load()` returns a **proxy** and throws `ObjectNotFoundException` when accessed if missing. (JPA equivalents: `find()` and `getReference()`.)

**64. What is the difference between `persist()`, `save()` and `merge()`?**

| Method | Behavior |
|---|---|
| `persist()` | JPA standard. Makes a transient entity managed, returns void |
| `save()` | Hibernate legacy. Returns the id (deprecated in newer Hibernate) |
| `merge()` | Copies a detached entity's state onto a managed instance and **returns the managed copy** |

**65. What is flushing?**
👉 Synchronizing the persistence context with the DB (running the pending INSERT/UPDATE/DELETE SQL). It happens before a query, at commit, or by calling `flush()`. Flush does **not** commit.

**66. How does dirty checking work internally?**
👉 When an entity is loaded, Hibernate keeps a **snapshot**. At flush, it compares the current state with the snapshot and generates UPDATE statements for the changed entities.
- Multiple setters on one entity result in **one UPDATE** at flush time, not one per setter.
- By default the UPDATE lists **all columns**. Use `@DynamicUpdate` to include only the changed ones.
- If the new value equals the old value, no UPDATE is sent (see Q17).

**67. What are the `ddl-auto` values?**

| Value | Behavior |
|---|---|
| `none` | Do nothing |
| `validate` | Check that the schema matches the entities |
| `update` | Update the schema (adds, doesn't drop) |
| `create` | Drop and recreate at startup |
| `create-drop` | Create at startup, drop at shutdown |

Production: use `validate` or `none` with migration tools (Flyway / Liquibase), never `update`/`create`.

**68. How do you see the SQL generated by Hibernate?**
```properties
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
logging.level.org.hibernate.orm.jdbc.bind=TRACE   # bound parameter values
```

**69. Does Hibernate have its own connection pool?**
👉 It has a basic built-in one that is **not for production**. Spring Boot uses **HikariCP** by default.

**70. What is the Criteria API?**
👉 A type-safe, programmatic way to build queries in Java code instead of strings. Useful for dynamic queries.

**71. How do you enable JDBC batching in Hibernate?**
```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```
It requires a generation strategy that allows it (not IDENTITY).

---

## 🔹 Transactions & Locking

**72. How does @Transactional work with Spring Data JPA?**
👉 Repository methods are already transactional (`SimpleJpaRepository`: `readOnly = true` for reads, a normal transaction for writes). Put `@Transactional` on the **service layer** so multiple repository calls run in one transaction. The transaction manager is `JpaTransactionManager`.

**73. What does `@Transactional(readOnly = true)` do in JPA?**
👉 Hibernate skips dirty checking and flushing for that session, which saves memory and CPU, and it can route to read replicas.

**74. What is Open Session in View (`spring.jpa.open-in-view`)?**
👉 It keeps the session open during the whole web request (default `true` in Spring Boot), so lazy loading works in controllers and views. The downside is that it holds a DB connection longer and can hide N+1 problems. Many teams set it to `false`.

**75. Optimistic vs pessimistic locking?**

| Optimistic | Pessimistic |
|---|---|
| Uses a `@Version` column | Locks the row in the DB (`SELECT ... FOR UPDATE`) |
| No lock held, conflict detected at update | Blocks other transactions |
| Best when conflicts are rare | Best when conflicts are frequent |

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT a FROM Account a WHERE a.id = :id")
Account findForUpdate(@Param("id") Long id);
```

---

## 🔹 Comparison & Scenario Questions

**76. JDBC vs Hibernate vs Spring Data JPA?**

| | JDBC | Hibernate | Spring Data JPA |
|---|---|---|---|
| Code | Most boilerplate | Less | Least |
| SQL | Manual | Auto (HQL/JPQL) | Auto (method names, `@Query`) |
| Control | Full | High | Lower |
| Mapping | Manual | Automatic | Automatic |

**77. What are the disadvantages of Hibernate / JPA?**
👉
- Learning curve (lifecycle, lazy loading, cascade)
- Hidden SQL and the N+1 problem
- Slower than raw SQL for bulk or reporting queries
- Less control over complex, DB-specific queries
- Memory overhead from the persistence context

**78. What should `equals()` and `hashCode()` look like in entities?**
👉 Avoid including mutable or lazy fields and avoid Lombok `@Data`. Use a stable business key, or an id-based implementation that handles a null id for new entities. It matters when entities are stored in a `Set`.

**79. Why prefer DTOs over returning entities from the controller?**
👉 Avoids exposing internal fields, prevents lazy-loading and infinite-recursion issues, controls the API shape, and lets you fetch only the columns you need.

**80. Scenario: an API is slow when listing departments with employees. What do you check?**
👉 Turn on SQL logging and look for N+1. Fix with `JOIN FETCH` / `@EntityGraph` / batch size, use pagination, and use a DTO projection if you don't need the whole entity.

**81. Scenario: a bulk insert of 50,000 records with `saveAll()` is slow. How do you optimize?**
👉
- Use SEQUENCE (not IDENTITY) and enable `hibernate.jdbc.batch_size`
- Insert in chunks and call `flush()` + `clear()` to free the persistence context
- Or use `JdbcTemplate.batchUpdate()` for very large loads

**82. Scenario: `save()` runs a SELECT before the INSERT. Why?**
👉 If you set the id manually, Spring Data thinks the entity is not new and calls `merge()`, which first does a SELECT. Let the DB generate the id, or implement `Persistable` / use a `@Version` field.

**83. Scenario: `LazyInitializationException` in a controller. What is your fix?**
👉 Load the needed association inside the service transaction (`JOIN FETCH` / `@EntityGraph`) or return a DTO. Don't depend on open-in-view.

**84. When would you use JdbcTemplate instead of JPA in the same project?**
👉 For heavy reports, complex joins/aggregations, bulk operations, and DB-specific SQL. Both can run in one project with a shared DataSource and `JpaTransactionManager`.

**85. How do you handle a unique-constraint violation (duplicate email)?**
👉 Add `unique = true` on the column (and a DB unique constraint). Catch `DataIntegrityViolationException` and return a meaningful 409 Conflict. Optionally check `existsByEmail()` first for a friendly message, but the DB constraint stays the real guard.

---

# PART C: Advanced Questions Expected in Interviews

## 🔹 Internals: How the Magic Works

**86. How does Spring Data JPA create the repository implementation when you only write an interface?**
👉 At startup Spring creates a **JDK dynamic proxy** for each repository interface (through `JpaRepositoryFactoryBean`).
- CRUD methods (`save`, `findById`...) are delegated to `SimpleJpaRepository`, the default implementation.
- Query methods are resolved once at startup: `@Query` first, then `@NamedQuery`, then a **derived query** parsed from the method name (`PartTree`). This is why an invalid method name fails at **startup**, not at runtime.

**87. How does Spring Boot auto-configure JPA and Hibernate?**
👉
- `DataSourceAutoConfiguration` → DataSource (HikariCP)
- `HibernateJpaAutoConfiguration` → `EntityManagerFactory`, `JpaTransactionManager`
- `JpaRepositoriesAutoConfiguration` → enables the repositories (like `@EnableJpaRepositories`)
- `@EntityScan` and `@EnableJpaRepositories(basePackages = ...)` are needed only when entities/repositories live **outside** the main application package.

**88. Is `EntityManager` thread-safe? How is it injected into singleton beans?**
👉 A raw `EntityManager` is **not** thread-safe. Spring injects a **shared EntityManager proxy** that delegates each call to the EntityManager bound to the **current transaction/thread**. So injecting it into a singleton with `@PersistenceContext` (or constructor injection in Boot) is safe.

**89. How does Hibernate implement lazy loading internally?**
👉 It creates a **proxy**, a runtime-generated **subclass** of your entity (ByteBuddy). The proxy holds only the id. On the first method call (other than `getId()`), it runs the SELECT and delegates to the real object.

Consequences:
- Entity classes and methods must **not be `final`**
- `getClass()` returns the proxy class, so use `instanceof` or `Hibernate.unproxy()`
- Use `Hibernate.initialize(obj.getItems())` to force loading inside a transaction

**90. Why doesn't lazy loading work on the non-owning side of a `@OneToOne`?**
👉 Hibernate must know whether the association is `null` or not to decide between a proxy and `null`. For the inverse side (`mappedBy`) it can't tell without a query, so it loads eagerly.

Fixes: use the owning side for lazy access, use **`@MapsId`** (shared primary key), or enable bytecode enhancement.

**91. What is the difference between `CascadeType.REMOVE`, `orphanRemoval` and `ON DELETE CASCADE`?**

| Option | Where it works | What it does |
|---|---|---|
| `CascadeType.REMOVE` | Hibernate/JPA | Deleting the **parent** deletes the children (loaded, then deleted one by one) |
| `orphanRemoval = true` | Hibernate/JPA | Removing a child from the parent's **collection** deletes it |
| `ON DELETE CASCADE` | **Database** foreign key | The DB deletes children itself. Fast, but Hibernate doesn't know |

---

## 🔹 Relationships & Fetching (Advanced)

**92. Why should you keep both sides of a bidirectional relationship in sync?**
👉 Hibernate only persists the **owning side**. If you update only one side, the object graph in memory becomes inconsistent. Use helper methods:

```java
public void addEmployee(Employee e) {
    employees.add(e);
    e.setDepartment(this);
}
public void removeEmployee(Employee e) {
    employees.remove(e);
    e.setDepartment(null);
}
```

**93. What happens with a unidirectional `@OneToMany` without `@JoinColumn`?**
👉 Hibernate creates an extra **join table** and does extra INSERT/DELETE/UPDATE statements, which is inefficient. Prefer a bidirectional mapping with `@ManyToOne` (owning side), or add `@JoinColumn` on the `@OneToMany`.

**94. What is `MultipleBagFetchException`?**
👉 Thrown when a query `JOIN FETCH`es **two or more `List` (bag) collections** at once. Fixes:
- Use `Set` instead of `List`
- Fetch the collections in **separate queries**
- Use `@OrderColumn` for ordered lists
- Use `@BatchSize` / `default_batch_fetch_size`

**95. Why is `JOIN FETCH` + pagination a problem?**
👉 Joining a collection multiplies rows, so the database can't limit by parent entity. Hibernate logs *"firstResult/maxResults specified with collection fetch; applying in memory"* (HHH90003004) and loads **everything** into memory, then paginates.

Fix: first page the **parent ids** (or parents without the fetch), then load children with a second query (`WHERE id IN (...)` + `JOIN FETCH`), or use `@BatchSize`.

**96. What is the Cartesian product problem in fetch joins?**
👉 Fetching multiple collections in one query returns every combination of rows (for example, 10 items × 10 tags = 100 rows per parent). Use separate queries or batch fetching instead.

**97. What are `@BatchSize` and `FetchMode.SUBSELECT`?**
👉 Both reduce the N+1 queries for lazy collections:
- `@BatchSize(size = 20)` → loads lazy collections for **20 parents at once** using `IN (?, ?, ...)`
- `@Fetch(FetchMode.SUBSELECT)` → loads the collections for all parents with a single subselect query

**98. How do you pick a Set vs a List for collections?**
👉 Use a `Set` for `@ManyToMany` (avoids the "bag" problem and makes removes efficient). A `List` without `@OrderColumn` is a *bag* with no order guarantee, and removing from it may delete and recreate all the link rows.

**99. What is a JPQL constructor expression?**
👉 Creates DTOs directly from a query, with no entity loaded:

```java
@Query("SELECT new com.app.dto.StudentDto(s.name, s.email) FROM Student s WHERE s.age > :age")
List<StudentDto> findDtos(@Param("age") int age);
```

**100. What is the difference between closed and open projections?**
👉
- **Closed:** the interface has getters that match entity properties. Spring selects **only those columns**.
- **Open:** uses `@Value("#{target.firstName + ' ' + target.lastName}")`. Spring has to load the **whole entity**, so there is no column optimization.

---

## 🔹 Persistence Context & Flush (Advanced)

**101. What are the flush modes and when does auto flush happen?**
👉
- `FlushModeType.AUTO` (default): Hibernate flushes before running a query that may be affected by pending changes, and at commit
- `FlushModeType.COMMIT`: flushes only at commit
- Spring sets the mode to **MANUAL** for `readOnly = true` transactions (no flush at all)

**102. In what order does Hibernate execute the SQL at flush time?**
👉 The **action queue** runs in this order: **inserts → updates → collection changes → deletes**, not the order of your Java code.

Gotcha: "delete a row, then insert a row with the same unique key" in one transaction can fail with a unique-constraint error, because the insert runs before the delete. Fix: call `flush()` after the delete.

**103. Does a JPQL query hit the first-level cache?**
👉 The **query itself always goes to the DB**. But when the rows come back, Hibernate checks the persistence context. If an entity with that id is already managed, it **returns the existing instance** (and ignores the fresh row data). Only `find()`/`findById()` can be served from the first-level cache without a query.

**104. How do you handle memory growth in a large batch job?**
👉 The persistence context keeps every managed entity. Process in chunks and call `flush()` then `clear()` (or `entityManager.detach()`).

```java
for (int i = 0; i < list.size(); i++) {
    em.persist(list.get(i));
    if (i % 50 == 0) {
        em.flush();
        em.clear();
    }
}
```

**105. What is a `StatelessSession`?**
👉 A Hibernate session with **no persistence context**: no first-level cache, no dirty checking, no cascades, no lazy loading. It is used for fast bulk inserts/updates and for streaming large data.

---

## 🔹 Caching

**106. How do you enable the second-level cache?**
👉
1. Add a cache provider (Ehcache / Caffeine via JCache / Infinispan)
2. Turn it on:
```properties
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```
3. Mark entities as cacheable:
```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Country { ... }
```
Strategies: `READ_ONLY`, `NONSTRICT_READ_WRITE`, `READ_WRITE`, `TRANSACTIONAL`.
Best for **read-mostly reference data** (countries, roles), not for frequently changing data.

**107. What is the query cache and when is it useful?**
👉 It caches **query results** (the list of ids) in the second-level cache. It needs `hibernate.cache.use_query_cache=true` and a `cacheable` query hint. Any change to a table involved **invalidates** the cached queries, so it helps only for read-mostly data.

---

## 🔹 Locking & Concurrency

**108. What is the lost update problem and how do you prevent it?**
👉 Two transactions read the same row, both modify, and the later write silently overwrites the earlier one.

Prevention:
- **Optimistic locking** with `@Version`
- **Pessimistic locking** (`PESSIMISTIC_WRITE`)
- Atomic update: `UPDATE account SET balance = balance - :amt WHERE id = :id`

**109. How do you handle `OptimisticLockException` in practice?**
👉 It means someone else changed the row. Options: return **409 Conflict** to the client and ask them to reload, or **retry** the operation (for example, with Spring Retry's `@Retryable(ObjectOptimisticLockingFailureException.class)`). Retry only if the whole operation can be safely repeated.

**110. What are the pessimistic lock modes?**
👉 `PESSIMISTIC_READ` (shared lock), `PESSIMISTIC_WRITE` (exclusive, `SELECT ... FOR UPDATE`), `PESSIMISTIC_FORCE_INCREMENT`. Set a lock timeout (`jakarta.persistence.lock.timeout`) to avoid long waits, and keep the transaction short to reduce **deadlocks**.

---

## 🔹 Mapping (Advanced)

**111. How do you map a composite primary key?**
👉 Two ways. The key class must be `Serializable` and implement `equals()` and `hashCode()`.

```java
// 1) @EmbeddedId: key fields live inside an @Embeddable class
@Embeddable
public class OrderItemId implements Serializable {
    private Long orderId;
    private Long productId;
}

@Entity
public class OrderItem {
    @EmbeddedId
    private OrderItemId id;
}

// 2) @IdClass: the fields are repeated in the entity, each marked @Id
```

**112. What is the SEQUENCE `allocationSize` and why does it matter?**
👉 Hibernate reserves a block of ids (default **50**) per DB call and hands them out from memory. The DB sequence's `INCREMENT BY` must match `allocationSize`, otherwise you get duplicate keys or big gaps. IDs can have gaps after restarts, which is normal.

**113. How do you implement soft delete?**
👉
```java
@Entity
@SQLDelete(sql = "UPDATE student SET deleted = true WHERE id = ?")
@SQLRestriction("deleted = false")   // Hibernate 6.3+ (older: @Where)
public class Student { private boolean deleted; }
```
`delete()` becomes an UPDATE, and normal entity queries hide deleted rows. Newer Hibernate also has `@SoftDelete`. Be careful: native queries do not apply the restriction automatically.

**114. What are JPA lifecycle callbacks?**
👉 Methods on the entity run at lifecycle events: `@PrePersist`, `@PostPersist`, `@PreUpdate`, `@PostUpdate`, `@PreRemove`, `@PostRemove`, `@PostLoad`. Typical use: set timestamps, validate, normalize data.

**115. How do you fill `createdBy` / `modifiedBy` automatically?**
👉 Use `@CreatedBy` / `@LastModifiedBy` and provide an `AuditorAware<String>` bean that returns the current user (for example, from Spring Security).

**116. What is Hibernate Envers?**
👉 An auditing module. Add `@Audited` to an entity and Envers keeps a history (`<table>_AUD` tables plus revision info) of every change, so you can query previous versions.

---

## 🔹 Queries & Performance (Advanced)

**117. Offset pagination vs keyset (scroll) pagination?**
👉
- **Offset** (`Pageable`): `LIMIT n OFFSET m`. Simple, but deep pages are slow because the DB skips `m` rows.
- **Keyset / scroll:** `WHERE id > :lastId ORDER BY id LIMIT n`. Constant speed for deep pages, no jumping to arbitrary pages. Spring Data JPA 3.1+ supports it through `Window<T>` / `ScrollPosition`.

**118. How do you stream or process a huge table with Spring Data JPA?**
👉 Return `Stream<T>` from the repository inside `@Transactional(readOnly = true)`, close the stream (try-with-resources), and set a fetch size.

```java
@QueryHints(@QueryHint(name = HINT_FETCH_SIZE, value = "500"))
Stream<Student> streamAllBy();
```
Caution: managed entities still pile up in the persistence context, so `detach()`/`clear()` periodically, or use a `StatelessSession`. Some drivers (MySQL) need extra settings for real streaming.

**119. How do you detect and fix slow queries / N+1 in a project?**
👉
- Enable SQL logging, or `hibernate.generate_statistics=true` to see query counts
- Log slow queries with `hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS`
- Use tools like p6spy, datasource-proxy, or APM
- Check the DB `EXPLAIN` plan and add the right **indexes**
- Fix with fetch joins, `@EntityGraph`, batch fetching, DTO projections

**120. How do you map a native query's result to a DTO?**
👉 Use an **interface projection** (columns must be aliased to getter names), or `@SqlResultSetMapping` with `@ConstructorResult`.

```java
@Query(value = "SELECT s.name AS name, s.email AS email FROM student s", nativeQuery = true)
List<StudentView> findNativeViews();
```

**121. How do you do pagination with a `@Query` that has a join?**
👉 Provide a separate `countQuery`, since Spring's auto-generated count query can be wrong or slow for complex queries.

```java
@Query(value = "SELECT s FROM Student s JOIN s.courses c WHERE c.name = :name",
       countQuery = "SELECT COUNT(s) FROM Student s JOIN s.courses c WHERE c.name = :name")
Page<Student> findByCourse(@Param("name") String name, Pageable pageable);
```

**122. How does `saveAll()` behave, and is it the best for bulk operations?**
👉 It calls `save()` for each entity in one transaction. Each entity goes through persist/merge and dirty checking, so with large volumes it is slow and memory-heavy. For very large loads use JDBC batching settings, `flush()`/`clear()` chunks, `JdbcTemplate.batchUpdate()`, or native bulk tools.

---

## 🔹 Configuration & Operations

**123. How do you configure multiple DataSources with JPA?**
👉 Create a DataSource, a `LocalContainerEntityManagerFactoryBean` and a `JpaTransactionManager` **for each database**, and point each repository package to its own pair.

```java
@Configuration
@EnableJpaRepositories(
    basePackages = "com.app.orders.repo",
    entityManagerFactoryRef = "ordersEmf",
    transactionManagerRef = "ordersTm")
class OrdersDbConfig { ... }
```
Use `@Transactional("ordersTm")` to pick the right transaction manager.

**124. Why use Flyway/Liquibase instead of `ddl-auto`?**
👉 They give **versioned, reviewable, repeatable** schema migrations for every environment. `ddl-auto=update` can't drop/rename columns, gives no history, and can behave unpredictably in production. Use Flyway/Liquibase and `ddl-auto=validate`.

**125. How do you test the repository layer?**
👉 `@DataJpaTest` loads only JPA components, uses an embedded DB by default, and rolls back after each test. For database-specific behavior use **Testcontainers** with `@AutoConfigureTestDatabase(replace = NONE)`.

**126. What is the difference between `@Transactional` rollback in tests and real behavior?**
👉 Tests roll back by default, so changes are **never flushed/committed**, and some problems (constraint violations, flush-time errors) do not show up. Call `flush()` in the test when you need the SQL to run.

---

## 🔹 Common Exceptions

**127. What exceptions are commonly seen with Spring Data JPA and what do they mean?**

| Exception | Meaning / Fix |
|---|---|
| `LazyInitializationException` | Lazy field accessed after the session closed. Fetch in the transaction |
| `DataIntegrityViolationException` | Unique/FK/NOT NULL violation |
| `ObjectOptimisticLockingFailureException` | `@Version` conflict. Reload or retry |
| `IncorrectResultSizeDataAccessException` (`NonUniqueResultException`) | Query expected one row but got more |
| `PropertyReferenceException` | Derived query uses a property that doesn't exist (`No property xyz found`) |
| `TransactionRequiredException` | `@Modifying` query or write without a transaction |
| `InvalidDataAccessApiUsageException` ("unsaved transient instance") | Referencing a child/parent that isn't saved yet (missing cascade or save) |
| `MultipleBagFetchException` | Fetching two `List` collections in one query |
| `PersistentObjectException` ("detached entity passed to persist") | Calling `persist` on an entity that already has an id (use `merge`/`save`) |

---

## 🔹 Scenario Questions

**128. Scenario: two users edit the same record at the same time. How do you avoid overwriting?**
👉 Add `@Version` (optimistic locking). The second save fails with `ObjectOptimisticLockingFailureException`. Return 409 and ask the user to refresh. For high-contention rows (like account balances), use pessimistic locking or an atomic UPDATE.

**129. Scenario: `orphanRemoval` is set but removing a child from the list doesn't delete it. Why?**
👉 Common causes: you replaced the collection with a **new list** (`setItems(new ArrayList<>())`) instead of modifying the existing one (Hibernate tracks the original collection instance), there is no transaction, or the child is still referenced elsewhere. Modify the existing collection with `remove()` / `clear()`.

**130. Scenario: the same data is being queried repeatedly and the DB is overloaded. What do you do?**
👉 First check indexes and N+1. Then add caching in layers: Spring Cache (`@Cacheable`) at the service level for results, and Hibernate second-level cache for read-mostly entities. Consider read replicas with `@Transactional(readOnly = true)` routing.

**131. Scenario: `findAll()` on a 10-million-row table crashed the app. How do you redesign it?**
👉 Never load everything. Use **pagination** (keyset for deep pages), **streaming** with a small fetch size and periodic `clear()`, a DTO projection with only the needed columns, or move the heavy processing into a DB query / batch job.

**132. Scenario: you must update 1 lakh (100,000) rows with the same change. Which approach?**
👉 A single **bulk JPQL/native UPDATE** (`@Modifying`) is by far the fastest, because loading entities and dirty checking would be slow. Call `clearAutomatically = true` afterwards so stale entities aren't used.

**133. Scenario: the response time grows as the database grows. Where do you look?**
👉 Missing indexes on filter/join columns, offset pagination on deep pages, N+1 queries, large result sets without projections, long transactions, and the connection pool size. Use `EXPLAIN` and Hibernate statistics to find the real bottleneck.
