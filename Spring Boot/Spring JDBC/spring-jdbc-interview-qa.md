# Spring JDBC Interview Questions & Answers

## 🔹 Basics

**1. What is Spring JDBC?**
👉 Spring JDBC is a Spring Framework module that provides a simplified and consistent way to interact with relational databases using JDBC, by handling resource management and translating SQL exceptions into runtime exceptions.

**2. Why do we use Spring JDBC instead of Plain JDBC?**
👉 Because Spring JDBC removes boilerplate code, manages resources automatically, and makes database access cleaner, safer, and easier to maintain.

**3. What problems exist in traditional JDBC?**
👉 It needs a lot of repetitive code:
- Opening/closing connections
- Handling SQLException
- Writing try-catch-finally blocks
- Mapping ResultSet to objects

Spring JDBC solves these problems while still letting you write SQL directly.

**4. Which dependency is used for Spring JDBC in Spring Boot?**
👉 `spring-boot-starter-jdbc`. It brings in Spring JDBC, HikariCP, and auto-configuration for `DataSource` and `JdbcTemplate`.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

**5. What are the main classes/components in Spring JDBC?**
👉 `JdbcTemplate`, `NamedParameterJdbcTemplate`, `SimpleJdbcInsert`, `SimpleJdbcCall`, `RowMapper`, `ResultSetExtractor`, `DataSource`.

---

## 🔹 DataSource & Configuration

**6. What is DataSource in Spring JDBC?**
👉 DataSource is an interface that provides database connections and manages connection pooling in Spring applications.

```java
public interface DataSource {
    Connection getConnection() throws SQLException;
}
```

**7. How is the DB connected in Spring Boot?**
👉 Spring Boot auto-configures `DataSource` and `JdbcTemplate`. No XML is required. You only provide properties:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/schooldb
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

**8. Explain `spring.datasource.url`.**
👉 `jdbc:mysql://localhost:3306/schooldb`
- `jdbc:mysql://` → JDBC protocol + MySQL database
- `localhost` → database is on the local machine
- `3306` → default MySQL port
- `schooldb` → database name

**9. What do `spring.datasource.username` and `password` do?**
👉 They give the credentials MySQL uses to authenticate the connection.
⚠️ **Best Practice:** Never hardcode passwords in production. Use environment variables or vaults.

**10. How do you configure DataSource manually? (Rare)**
👉 Using `DriverManagerDataSource` in a `@Bean`:

```java
@Bean
public DataSource dataSource() {
    DriverManagerDataSource ds = new DriverManagerDataSource();
    ds.setUrl("jdbc:mysql://localhost:3306/schooldb");
    ds.setUsername("root");
    ds.setPassword("root");
    return ds;
}
```

**11. What is the difference between DriverManagerDataSource and a pooled DataSource (HikariCP)?**
👉 `DriverManagerDataSource` creates a new connection on every request and has no pooling, so it is only for testing or small demos. HikariCP reuses connections from a pool and is used in production.

**12. What is connection pooling?**
👉 Reusing database connections instead of creating new ones every time.

**13. What is the default connection pool in Spring Boot?**
👉 HikariCP.

---

## 🔹 JdbcTemplate

**14. What is JdbcTemplate in Spring JDBC?**
👉 JdbcTemplate is the core Spring JDBC class that simplifies database access by handling connection management, SQL execution, and exception handling.

**15. What are the main responsibilities of JdbcTemplate?**
👉 Creating connections, executing SQL, mapping results, and handling exceptions.

**16. How does JdbcTemplate get the database connection?**
👉 It uses a `DataSource` bean that Spring Boot auto-configures from the application properties.

**17. Why JdbcTemplate?**
👉 Before Spring JDBC, we had to open connections, create statements, handle exceptions, and close resources manually 😓. JdbcTemplate does all of this automatically.

**18. What are the common JdbcTemplate methods?**

| Method | Purpose |
|---|---|
| `update()` | INSERT / UPDATE / DELETE |
| `query()` | SELECT multiple rows |
| `queryForObject()` | SELECT single row |
| `queryForList()` | List of rows |
| `queryForMap()` | Single row as Map |
| `batchUpdate()` | Batch operations |

**19. Write an INSERT example using JdbcTemplate.**
```java
@Repository
public class StudentRepository {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public int save(Student s) {
        String sql = "INSERT INTO student(id, name, email) VALUES (?, ?, ?)";
        return jdbcTemplate.update(sql, s.getId(), s.getName(), s.getEmail());
    }
}
```

**20. Write a SELECT example to fetch all students.**
```java
public List<Student> getAllStudents() {
    return jdbcTemplate.query(
        "SELECT * FROM student",
        new StudentRowMapper()
    );
}
```

**21. What is the difference between `update()` and `batchUpdate()`?**
👉 `update()` executes one SQL statement per call. `batchUpdate()` sends multiple statements together in one batch, which cuts DB round trips and is much faster for bulk inserts/updates.

**22. What is the difference between `query()`, `queryForObject()`, and `queryForList()`?**
👉
- `query()` → multiple rows, mapped with RowMapper/ResultSetExtractor
- `queryForObject()` → exactly one row (or a single value like `count(*)`)
- `queryForList()` → list of rows as `Map<String, Object>` or a list of simple values

**23. What happens if `queryForObject()` returns multiple rows?**
👉 It throws `IncorrectResultSizeDataAccessException`.

**24. What happens if `queryForObject()` returns zero rows?**
👉 It throws `EmptyResultDataAccessException`.

**25. Is JdbcTemplate thread-safe?**
👉 Yes, because it does not store connection state internally.

**26. Does JdbcTemplate use Statement or PreparedStatement?**
👉 It uses `PreparedStatement` internally.
📌 **Trick:** This is why SQL injection is prevented.

**27. How do you get the auto-generated key after an INSERT?**
👉 Using `KeyHolder` with `GeneratedKeyHolder`:

```java
KeyHolder keyHolder = new GeneratedKeyHolder();
jdbcTemplate.update(con -> {
    PreparedStatement ps = con.prepareStatement(
        "INSERT INTO student(name, email) VALUES (?, ?)",
        Statement.RETURN_GENERATED_KEYS);
    ps.setString(1, s.getName());
    ps.setString(2, s.getEmail());
    return ps;
}, keyHolder);

Number id = keyHolder.getKey();
```

**28. What is SimpleJdbcInsert?**
👉 A helper class that builds the INSERT statement automatically from table metadata, so you don't write the SQL.

```java
SimpleJdbcInsert insert = new SimpleJdbcInsert(dataSource)
        .withTableName("student");
insert.execute(Map.of("id", 1, "name", "Harish"));
```

**29. How do you call a stored procedure in Spring JDBC?**
👉 Using `SimpleJdbcCall` (or `JdbcTemplate.call()`).

```java
SimpleJdbcCall call = new SimpleJdbcCall(dataSource)
        .withProcedureName("get_student");
Map<String, Object> result = call.execute(Map.of("p_id", 1));
```

---

## 🔹 RowMapper & ResultSetExtractor

**30. What is RowMapper in Spring JDBC?**
👉 RowMapper is an interface used to convert each row of a ResultSet into a Java object. Method: `mapRow()`.

**31. What is ResultSetExtractor in Spring JDBC?**
👉 ResultSetExtractor is an interface used to extract data from the entire ResultSet and return any custom object. Method: `extractData()`.

**32. What is the difference between RowMapper and ResultSetExtractor?**

| RowMapper | ResultSetExtractor |
|---|---|
| Called once per row | Called once for the whole ResultSet |
| Method: `mapRow()` | Method: `extractData()` |
| Returns one object per row | Returns any custom object (List, Map, nested object) |
| Spring iterates the ResultSet | You iterate the ResultSet yourself |

🎯 **Interview Line:** Use RowMapper for simple row-to-object mapping and ResultSetExtractor for complex mappings such as one-to-many joins.

**33. Write a RowMapper implementation.**
```java
public class StudentRowMapper implements RowMapper<Student> {
    @Override
    public Student mapRow(ResultSet rs, int rowNum) throws SQLException {
        Student s = new Student();
        s.setId(rs.getInt("id"));
        s.setName(rs.getString("name"));
        s.setEmail(rs.getString("email"));
        return s;
    }
}
```

**34. What is BeanPropertyRowMapper?**
👉 A ready-made RowMapper that maps columns to bean properties by name automatically, so you don't write `mapRow()`.

```java
jdbcTemplate.query("SELECT * FROM student",
    new BeanPropertyRowMapper<>(Student.class));
```

---

## 🔹 NamedParameterJdbcTemplate

**35. What is NamedParameterJdbcTemplate in Spring JDBC?**
👉 It is a Spring JDBC class that allows you to use named parameters in SQL instead of positional `?` placeholders.

**36. Why use NamedParameterJdbcTemplate over JdbcTemplate?**
👉 Named parameters (`:id`, `:name`) are more readable and less error-prone than `?`, especially when a query has many parameters, because there is no need to track positions.

**37. Write an INSERT example using NamedParameterJdbcTemplate.**
```java
@Repository
public class StudentRepository {

    @Autowired
    private NamedParameterJdbcTemplate namedJdbcTemplate;

    public int save(Student s) {
        String sql = "INSERT INTO student (id, name, email) " +
                     "VALUES (:id, :name, :email)";

        Map<String, Object> params = new HashMap<>();
        params.put("id", s.getId());
        params.put("name", s.getName());
        params.put("email", s.getEmail());

        return namedJdbcTemplate.update(sql, params);
    }
}
```

**38. What is BeanPropertySqlParameterSource?**
👉 It automatically maps object fields to named parameters.

```java
SqlParameterSource params = new BeanPropertySqlParameterSource(s);
return namedJdbcTemplate.update(sql, params);
```

**39. What is MapSqlParameterSource?**
👉 We manually provide the values:

```java
SqlParameterSource params =
    new MapSqlParameterSource()
        .addValue("id", 101)
        .addValue("name", "Harish");
```

**40. Write a SELECT example using NamedParameterJdbcTemplate.**
```java
public Student findById(int id) {
    String sql = "SELECT * FROM student WHERE id = :id";
    Map<String, Object> params = Map.of("id", id);
    return namedJdbcTemplate.queryForObject(
        sql, params, new StudentRowMapper());
}
```

---

## 🔹 Exception Handling

**41. How does Spring JDBC handle exceptions?**
👉 Spring converts `SQLException` into unchecked exceptions, so there is no need to write `throws SQLException`:
- `DataAccessException` (root)
- `DuplicateKeyException`
- `EmptyResultDataAccessException`

**42. Why is DataAccessException unchecked?**
👉 So service/repository code isn't forced into try-catch or `throws` clauses. It is also database-independent: Spring translates vendor-specific error codes into a common hierarchy.

---

## 🔹 @Transactional

**43. What is @Transactional?**
👉 A Spring annotation used to manage database transactions automatically by committing on success and rolling back on failure.

**44. What are the important attributes of @Transactional?**

| Attribute | Meaning |
|---|---|
| `propagation` | How transaction behaves |
| `isolation` | Data visibility |
| `rollbackFor` | Rollback for checked exceptions |
| `readOnly` | Optimization for SELECT |
| `timeout` | Max execution time |

**45. What is propagation in @Transactional?**
👉 It defines how a method joins or creates a transaction. Default: `Propagation.REQUIRED`.

| Type | Meaning |
|---|---|
| REQUIRED | Join existing or create new |
| REQUIRES_NEW | Always create new |
| SUPPORTS | Join if exists |
| NOT_SUPPORTED | Execute without transaction |
| MANDATORY | Must have transaction |
| NEVER | Must not have transaction |

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void savePayment() { }
```
🎯 **Interview Line:** Propagation defines how transactions behave when one method calls another.

**46. What is isolation in @Transactional?**
👉 It controls how one transaction sees changes from another. Default: `Isolation.DEFAULT` (uses the database default).

| Level | Problem Solved |
|---|---|
| READ_UNCOMMITTED | Allows dirty reads |
| READ_COMMITTED | Prevents dirty reads |
| REPEATABLE_READ | Prevents non-repeatable reads |
| SERIALIZABLE | Prevents all anomalies |

```java
@Transactional(isolation = Isolation.READ_COMMITTED)
```
🎯 **Interview Line:** Isolation controls concurrency issues like dirty reads and phantom reads.

**47. What is rollbackFor?**
👉 By default, Spring rolls back only for runtime exceptions. `rollbackFor` forces rollback even for checked exceptions.

```java
@Transactional(rollbackFor = Exception.class)
```

**48. What is readOnly in @Transactional?**
👉 It tells Spring this transaction won't modify data. Default: `false`.
✔️ Better performance ✔️ Prevents accidental writes

```java
@Transactional(readOnly = true)
public List<Student> getAllStudents() { }
```

**49. What is timeout in @Transactional?**
👉 It sets how long a transaction can run (in seconds) before rollback.

```java
@Transactional(timeout = 5)
public void processPayment() { }
```

**50. Show all attributes in one example.**
```java
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    rollbackFor = Exception.class,
    readOnly = false,
    timeout = 10
)
public void saveStudent() {
    // DB logic
}
```

**51. Which transaction manager is used with Spring JDBC?**
👉 `DataSourceTransactionManager`. Spring Boot auto-configures it when you use `spring-boot-starter-jdbc`.

**52. How does @Transactional work internally?**
👉 Through Spring AOP proxies. Spring wraps the bean in a proxy that begins the transaction before the method, commits on success, and rolls back on exception.

**53. Why does @Transactional not work when a method is called from the same class (self-invocation)?**
👉 The call bypasses the proxy, so the transaction logic never runs. Fix: move the method to another bean, or inject the proxy/self-reference.

---

## 🔹 Spring JDBC vs Spring Data JPA

**54. Which is best: Spring JDBC or Spring Data JPA?**
👉 Neither is universally "best". The choice depends on the use case. Spring JDBC is preferred for performance and SQL control, while Spring Data JPA is preferred for rapid development and complex domain models.

**55. Why is Spring JDBC preferred for performance and SQL control?**
👉
- **Performance:** raw SQL runs directly, with no ORM layer, no entity tracking, and no dirty checking. That means less CPU and memory, and it is faster for simple and bulk operations.
- **SQL control:** you write the exact SQL and control JOINs, index hints, database-specific functions, and stored procedures.

🎯 **Interview Line:** "Spring JDBC lets me optimize queries exactly as the database expects."

**56. Why is Spring Data JPA preferred for rapid development and complex domain models?**
👉
- **Rapid development:** minimal code, no SQL for basic CRUD, and auto-generated queries. A single `interface StudentRepository extends JpaRepository<Student, Long> {}` gives `save()`, `findById()`, `findAll()`, `delete()`.
- **Complex domain models:** automatic table-to-object mapping, with One-to-Many, Many-to-Many, cascading, and lazy loading handled for you.

**57. Give a real-world analogy for Spring JDBC vs JPA.**
👉
- **Spring JDBC = Manual car** → more control, better performance, requires skill
- **Spring Data JPA = Automatic car** → easier to drive, faster to build, less control

**58. When should you choose Spring JDBC, and when Spring Data JPA?**
👉
- **Spring JDBC:** performance is critical, heavy read/write operations, SQL optimization matters, reporting and financial systems.
- **Spring Data JPA:** complex business logic, many entity relationships, time-to-market is important, team productivity matters.

**59. How does Spring JDBC prevent SQL injection?**
👉 By using `PreparedStatement` with `?` or named placeholders, so values are bound as parameters and never concatenated into the SQL string.
⚠️ Never build SQL with string concatenation of user input.

---

# ➕ Extra Questions (Commonly Expected in Interviews)

## 🔹 JDBC Fundamentals

**60. What are the steps in plain JDBC?**
👉
1. Load the driver (automatic in modern JDBC)
2. Get a `Connection`
3. Create a `Statement` / `PreparedStatement`
4. Execute the query
5. Process the `ResultSet`
6. Close `ResultSet`, `Statement`, `Connection` (in `finally` or try-with-resources)

Spring JDBC automates steps 2, 3, 4 and 6.

**61. What is the difference between Statement, PreparedStatement and CallableStatement?**

| Type | Use |
|---|---|
| `Statement` | Static SQL, no parameters, open to SQL injection |
| `PreparedStatement` | Precompiled SQL with `?` parameters, safer and faster when reused |
| `CallableStatement` | Calling stored procedures / functions |

**62. What design pattern does JdbcTemplate follow?**
👉 The **Template Method** pattern. JdbcTemplate fixes the steps (get connection, create statement, execute, close, translate exceptions), and you only supply the variable parts through callbacks (`RowMapper`, `PreparedStatementSetter`, `ResultSetExtractor`).

🎯 **Interview Line:** "JdbcTemplate owns the boilerplate, I own the SQL and the mapping."

**63. What are the callback interfaces used by JdbcTemplate?**
👉
- `PreparedStatementCreator` → creates the PreparedStatement
- `PreparedStatementSetter` → sets parameter values
- `RowMapper` → maps one row
- `ResultSetExtractor` → processes the whole ResultSet
- `RowCallbackHandler` → processes each row with no return value
- `CallableStatementCreator` → creates the CallableStatement

**64. How does Spring JDBC avoid connection leaks?**
👉 JdbcTemplate always releases the connection in a `finally` block through `DataSourceUtils.releaseConnection()`. Inside a transaction, the same connection is bound to the thread and reused, then released when the transaction ends.

---

## 🔹 More JdbcTemplate Usage

**65. How do you fetch a single value like a count?**
```java
Integer count = jdbcTemplate.queryForObject(
    "SELECT COUNT(*) FROM student", Integer.class);
```

**66. Write UPDATE and DELETE examples with JdbcTemplate.**
```java
public int updateEmail(int id, String email) {
    return jdbcTemplate.update(
        "UPDATE student SET email = ? WHERE id = ?", email, id);
}

public int delete(int id) {
    return jdbcTemplate.update("DELETE FROM student WHERE id = ?", id);
}
```
`update()` returns the number of rows affected.

**67. Write a `queryForObject()` example for one student by id.**
```java
public Student findById(int id) {
    return jdbcTemplate.queryForObject(
        "SELECT * FROM student WHERE id = ?",
        new StudentRowMapper(), id);
}
```

**68. Write a `batchUpdate()` example.**
```java
public int[] saveAll(List<Student> students) {
    String sql = "INSERT INTO student(id, name, email) VALUES (?, ?, ?)";
    return jdbcTemplate.batchUpdate(sql, students, 100,
        (ps, s) -> {
            ps.setInt(1, s.getId());
            ps.setString(2, s.getName());
            ps.setString(3, s.getEmail());
        });
}
```
The `100` is the batch size. The result is an array of update counts.

**69. How do you handle a "not found" case with `queryForObject()`?**
👉 Catch `EmptyResultDataAccessException`, or use `query()` and check whether the list is empty (cleaner, no exception).

```java
List<Student> list = jdbcTemplate.query(sql, new StudentRowMapper(), id);
return list.isEmpty() ? Optional.empty() : Optional.of(list.get(0));
```

**70. What is `execute()` in JdbcTemplate?**
👉 A general-purpose method for any SQL, mostly DDL (e.g., `CREATE TABLE`), or when you need full control through a callback.

```java
jdbcTemplate.execute("CREATE TABLE test(id INT, name VARCHAR(50))");
```

**71. How do you implement pagination with Spring JDBC?**
👉 Use the database's pagination SQL.

```java
String sql = "SELECT * FROM student ORDER BY id LIMIT ? OFFSET ?";
List<Student> page = jdbcTemplate.query(sql, new StudentRowMapper(), size, page * size);
```
(Oracle uses `OFFSET ... FETCH NEXT`, older SQL Server uses `TOP`/`ROW_NUMBER`.)

**72. How do you process a very large result set without loading everything into memory?**
👉 Use `RowCallbackHandler` (process each row as it arrives) or `queryForStream()`, and set `setFetchSize()`.

```java
jdbcTemplate.query("SELECT * FROM big_table", rs -> {
    // process one row at a time
});
```
`queryForStream()` must be used in try-with-resources so the stream is closed.

**73. What is the difference between RowMapper and RowCallbackHandler?**
👉 `RowMapper` returns an object per row, and Spring builds the result list. `RowCallbackHandler` returns nothing (`void`), so you handle each row yourself. It is good for streaming or aggregating.

**74. How do you set query timeout, fetch size and max rows?**
```java
jdbcTemplate.setQueryTimeout(10);  // seconds
jdbcTemplate.setFetchSize(500);
jdbcTemplate.setMaxRows(1000);
```

**75. What is JdbcClient? (Spring 6.1 / Boot 3.2+)**
👉 A newer fluent API that combines the features of `JdbcTemplate` and `NamedParameterJdbcTemplate`.

```java
Student s = jdbcClient.sql("SELECT * FROM student WHERE id = :id")
        .param("id", 1)
        .query(Student.class)
        .single();
```

---

## 🔹 NamedParameterJdbcTemplate & Simple* Helpers

**76. How do you use an IN clause with NamedParameterJdbcTemplate?**
👉 Pass a collection as the parameter and Spring expands it automatically.

```java
String sql = "SELECT * FROM student WHERE id IN (:ids)";
Map<String, Object> params = Map.of("ids", List.of(1, 2, 3));
List<Student> list = namedJdbcTemplate.query(sql, params, new StudentRowMapper());
```
With plain `JdbcTemplate` you would have to build the `?,?,?` placeholders yourself.

**77. What is the relationship between NamedParameterJdbcTemplate and JdbcTemplate?**
👉 NamedParameterJdbcTemplate wraps a `JdbcTemplate`. It converts `:name` placeholders into `?` and then delegates to the JdbcTemplate.

**78. How do you get the generated key with SimpleJdbcInsert?**
```java
SimpleJdbcInsert insert = new SimpleJdbcInsert(dataSource)
        .withTableName("student")
        .usingGeneratedKeyColumns("id");
Number id = insert.executeAndReturnKey(Map.of("name", "Harish", "email", "h@x.com"));
```

**79. How do you get the generated key with NamedParameterJdbcTemplate?**
```java
KeyHolder keyHolder = new GeneratedKeyHolder();
namedJdbcTemplate.update(
    "INSERT INTO student(name, email) VALUES (:name, :email)",
    new BeanPropertySqlParameterSource(s), keyHolder, new String[]{"id"});
Number id = keyHolder.getKey();
```

---

## 🔹 RowMapper & ResultSetExtractor (Advanced)

**80. How do you map a one-to-many join using ResultSetExtractor?**
👉 Iterate the ResultSet yourself and group child rows under the parent.

```java
String sql = "SELECT o.id, o.total, i.id AS item_id, i.name " +
             "FROM orders o JOIN items i ON i.order_id = o.id";

Map<Integer, Order> orders = jdbcTemplate.query(sql, rs -> {
    Map<Integer, Order> map = new LinkedHashMap<>();
    while (rs.next()) {
        int id = rs.getInt("id");
        Order o = map.computeIfAbsent(id, k -> new Order(k));
        o.getItems().add(new Item(rs.getInt("item_id"), rs.getString("name")));
    }
    return map;
});
```
This is why a plain `RowMapper` is not enough for one-to-many.

**81. What is the difference between `BeanPropertyRowMapper` and `DataClassRowMapper`?**
👉 `BeanPropertyRowMapper` needs a default constructor and setters. `DataClassRowMapper` (Spring 5.3+) works with records and immutable classes through constructor mapping. Both map `snake_case` columns to `camelCase` fields automatically.

**82. What is the drawback of BeanPropertyRowMapper?**
👉 It uses reflection, so it is slightly slower than a hand-written RowMapper, and column names must match the property names. A custom RowMapper is faster and gives full control.

**83. How do you handle NULL values in a RowMapper?**
👉 For primitives, `rs.getInt()` returns `0` for NULL, so check `rs.wasNull()` or use `rs.getObject("col", Integer.class)`, which returns `null`.

---

## 🔹 DataSource, HikariCP & Configuration

**84. What are the important HikariCP properties?**
```properties
spring.datasource.hikari.maximum-pool-size=10     # default 10
spring.datasource.hikari.minimum-idle=10          # default = max pool size
spring.datasource.hikari.connection-timeout=30000 # ms, default 30s
spring.datasource.hikari.idle-timeout=600000      # ms, default 10 min
spring.datasource.hikari.max-lifetime=1800000     # ms, default 30 min
```

**85. How do you choose the pool size?**
👉 More is not better. A pool that is too large causes contention on the DB. Start small (around `(CPU cores * 2) + disk spindles` as a rough guide), then load test and tune.

**86. What happens when all connections in the pool are busy?**
👉 New requests wait up to `connection-timeout`. After that, Hikari throws a `SQLTransientConnectionException` ("Connection is not available, request timed out").

**87. How do you configure multiple DataSources?**
👉 Define multiple `DataSource` beans, mark one `@Primary`, and create a `JdbcTemplate` for each with `@Qualifier`.

```java
@Bean @Primary
@ConfigurationProperties("spring.datasource.primary")
public DataSource primaryDs() { return DataSourceBuilder.create().build(); }

@Bean
@ConfigurationProperties("spring.datasource.secondary")
public DataSource secondaryDs() { return DataSourceBuilder.create().build(); }

@Bean
public JdbcTemplate secondaryJdbc(@Qualifier("secondaryDs") DataSource ds) {
    return new JdbcTemplate(ds);
}
```
Each DataSource also needs its own transaction manager.

**88. Which Spring Boot auto-configuration classes set up JDBC?**
👉 `DataSourceAutoConfiguration` (DataSource), `JdbcTemplateAutoConfiguration` (JdbcTemplate and NamedParameterJdbcTemplate), and `DataSourceTransactionManagerAutoConfiguration` (transaction manager).

**89. How do you run schema.sql and data.sql at startup?**
👉 Put them in `src/main/resources`. Spring Boot runs them for embedded databases by default. For other databases, set:

```properties
spring.sql.init.mode=always
```

**90. How do you log the SQL executed by JdbcTemplate?**
```properties
logging.level.org.springframework.jdbc.core.JdbcTemplate=DEBUG
logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
```
The first logs the SQL, and the second logs the bound parameter values.

---

## 🔹 Exception Handling (Advanced)

**91. Explain the DataAccessException hierarchy.**
👉
- `DataAccessException` (root, unchecked)
  - `DataIntegrityViolationException` → constraint violations
    - `DuplicateKeyException` → unique key violation
  - `BadSqlGrammarException` → invalid SQL
  - `EmptyResultDataAccessException` / `IncorrectResultSizeDataAccessException`
  - `CannotGetJdbcConnectionException` → no connection
  - `DeadlockLoserDataAccessException` → deadlock
  - `QueryTimeoutException` → query timed out

**92. How does Spring translate SQLException into DataAccessException?**
👉 Through `SQLExceptionTranslator`. `SQLErrorCodeSQLExceptionTranslator` uses vendor error codes (from `sql-error-codes.xml`). If those are unavailable, `SQLStateSQLExceptionTranslator` uses the SQLState.

**93. What are transient and non-transient data access exceptions?**
👉
- **Transient:** may succeed if retried (deadlock, timeout, connection failure)
- **Non-transient:** retry will fail again (bad SQL, constraint violation)

**94. How do you handle DuplicateKeyException?**
```java
try {
    repo.save(student);
} catch (DuplicateKeyException e) {
    throw new StudentAlreadyExistsException(student.getId());
}
```

**95. What is the role of @Repository in Spring JDBC?**
👉 It marks the class as a DAO/persistence component and is a `@Component` stereotype. It also enables exception translation for repositories that use other persistence APIs through `PersistenceExceptionTranslationPostProcessor`. JdbcTemplate already translates exceptions on its own.

---

## 🔹 @Transactional & Transactions (Advanced)

**96. What are ACID properties?**
👉
- **Atomicity** → all or nothing
- **Consistency** → data moves from one valid state to another
- **Isolation** → concurrent transactions don't interfere
- **Durability** → committed data survives failures

**97. What are dirty read, non-repeatable read and phantom read?**

| Problem | Meaning |
|---|---|
| Dirty read | Reading uncommitted data from another transaction |
| Non-repeatable read | Same row read twice gives different values (another transaction updated it) |
| Phantom read | Same query run twice returns different sets of rows (another transaction inserted/deleted) |

**98. Which isolation level prevents which problem?**

| Level | Dirty | Non-repeatable | Phantom |
|---|---|---|---|
| READ_UNCOMMITTED | ❌ possible | ❌ possible | ❌ possible |
| READ_COMMITTED | ✅ prevented | ❌ possible | ❌ possible |
| REPEATABLE_READ | ✅ | ✅ | ❌ possible* |
| SERIALIZABLE | ✅ | ✅ | ✅ |

*MySQL InnoDB's default REPEATABLE_READ largely prevents phantoms as well.

**99. Explain REQUIRED vs REQUIRES_NEW with an example.**
👉
- `REQUIRED` → inner method joins the caller's transaction. If either fails, everything rolls back.
- `REQUIRES_NEW` → inner method suspends the caller's transaction and runs in its own. It commits or rolls back independently.

```java
@Transactional
public void placeOrder() {
    orderRepo.save(order);
    auditService.log("order placed");   // @Transactional(REQUIRES_NEW)
    // if placeOrder fails later, the audit log is still committed
}
```
Typical use: audit logging that must survive a rollback.

**100. What is NESTED propagation?**
👉 It runs inside the existing transaction using a **savepoint**. If the nested part fails, it rolls back to the savepoint only, while the outer transaction can continue. It needs JDBC savepoint support from the database.

**101. What is the default rollback behavior of @Transactional?**
👉 Rolls back on `RuntimeException` and `Error`. It does **not** roll back on checked exceptions unless `rollbackFor` is set.

**102. Why doesn't the transaction roll back when I catch the exception inside the method?**
👉 Spring only sees an exception that leaves the method. If you swallow it with try-catch, the proxy thinks the method succeeded and commits. Rethrow the exception, or call `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`.

**103. Does @Transactional work on private methods?**
👉 No. The proxy can only intercept calls to public methods from outside the bean. Use it on public methods.

**104. @Transactional on class level vs method level?**
👉 Class level applies to all public methods. Method level overrides the class level for that method. Best practice: put it on the **service layer**, not the repository or controller.

**105. What is TransactionTemplate? (Programmatic transactions)**
👉 A way to manage transactions in code, for fine-grained control.

```java
transactionTemplate.execute(status -> {
    studentRepo.save(s1);
    studentRepo.save(s2);
    return null;
});
```

**106. Declarative vs programmatic transaction management?**

| Declarative | Programmatic |
|---|---|
| `@Transactional` (AOP based) | `TransactionTemplate` / `PlatformTransactionManager` |
| Less code, cleaner | More code, more control |
| Preferred in most cases | Used for complex, conditional transactions |

**107. What is PlatformTransactionManager?**
👉 The central Spring interface for transaction management. Implementations include `DataSourceTransactionManager` (JDBC), `JpaTransactionManager` (JPA) and `HibernateTransactionManager`.

**108. Can JdbcTemplate and JPA be used together in the same transaction?**
👉 Yes, with `JpaTransactionManager`. It supports JDBC access on the same connection, as long as both use the same DataSource.

**109. Does readOnly = true really prevent writes?**
👉 Not always. It is a hint to the framework and driver. It can enable optimizations and some databases reject writes, but it is not a guaranteed safety lock.

---

## 🔹 Testing, Practical & Scenario Questions

**110. How do you test Spring JDBC code?**
👉 Use `@JdbcTest` with an embedded database (H2) for slice tests, or Testcontainers for a real database. Tests are rolled back automatically by default.

```java
@JdbcTest
class StudentRepositoryTest {
    @Autowired JdbcTemplate jdbcTemplate;
}
```

**111. What are the disadvantages of Spring JDBC?**
👉
- More SQL and mapping code to write
- No automatic object-relationship handling
- No caching, lazy loading or dirty checking
- SQL can be database-specific, which hurts portability
- Schema changes mean updating SQL and mappers manually

**112. How do you avoid SQL injection when the table or column name is dynamic?**
👉 Placeholders only work for **values**, not identifiers. For dynamic table/column names, validate against a **whitelist** before building the SQL.

**113. Scenario: a bulk insert of 1 lakh (100,000) records is slow. How do you optimize it?**
👉
- Use `batchUpdate()` with a suitable batch size (500–1000)
- Wrap it in one `@Transactional`
- For MySQL, add `rewriteBatchedStatements=true` to the JDBC URL
- Drop or defer indexes if it is a one-time load
- Consider native bulk tools (`LOAD DATA`, `COPY`) for huge loads

**114. Scenario: `Connection is not available, request timed out` error. What do you check?**
👉
- Connection leaks (connections not returned, e.g., manual `getConnection()` without close)
- Long-running transactions or slow queries holding connections
- Pool size too small for the load
- Remote calls made inside a `@Transactional` method
- Use Hikari's `leak-detection-threshold` to find leaks

**115. Scenario: how do you handle a deadlock?**
👉 Keep transactions short, access tables/rows in a consistent order, and retry on `DeadlockLoserDataAccessException` (for example, with Spring Retry).

**116. Scenario: how do you do an optimistic lock with Spring JDBC?**
👉 Add a `version` column and check it in the UPDATE:

```java
int rows = jdbcTemplate.update(
    "UPDATE student SET name = ?, version = version + 1 WHERE id = ? AND version = ?",
    name, id, version);
if (rows == 0) throw new OptimisticLockingFailureException("Stale data");
```

**117. Can we use Spring JDBC without Spring Boot?**
👉 Yes. Add `spring-jdbc`, define `DataSource` and `JdbcTemplate` beans manually, and add a `DataSourceTransactionManager` plus `@EnableTransactionManagement` for transactions.

**118. Spring JDBC vs Hibernate: which would you pick for a reporting module?**
👉 Spring JDBC. Reports need complex joins, aggregations and tuned SQL, and don't need entity management, so raw SQL gives better performance and control.
