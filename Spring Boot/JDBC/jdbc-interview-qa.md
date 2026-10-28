# JDBC Interview Questions & Answers

> Part A (Q1–15) is from your notes. Part B (Q16 onwards) is extra questions commonly asked in interviews.

# PART A: From Your Notes

## 🔹 Basics

**1. What is JDBC?**
👉 JDBC is a Java API that allows a Java application to connect with a database. It is used to execute SQL queries and retrieve the results.
💡 Think: JDBC is a **bridge** between the Java application and the database.

**2. What are the JDBC architecture layers?**
👉
```
Java Application
      ↓
   JDBC API
      ↓
JDBC DriverManager
      ↓
   Database
```

**3. What are the steps to connect a database to a Java application?**
👉 There are 7 steps:
1. Import packages (`java.sql.*`)
2. Load the driver
3. Register the driver
4. Create the connection
5. Create the statement
6. Execute the statement (`Statement` / `PreparedStatement`)
7. Close the resources

**4. Describe the JDBC connection flow and the purpose of the PostgreSQL JDBC driver.**
👉
1. **Add the PostgreSQL JDBC driver (external JAR).** `postgresql-42.x.x.jar` must be in the classpath. It contains the implementation of the JDBC interfaces for PostgreSQL. Without this JAR, Java cannot connect to PostgreSQL.
2. **Driver loading.** `Class.forName("org.postgresql.Driver");` is optional in JDBC 4+. It registers the driver with `DriverManager`. In modern Java this happens automatically using SPI (Service Provider Interface).
3. **DriverManager searches for a matching driver.**
   ```java
   Connection con = DriverManager.getConnection(
       "jdbc:postgresql://localhost:5432/testdb",
       "postgres",
       "password");
   ```
   `DriverManager` scans all loaded drivers, finds the one that understands the `jdbc:postgresql:` URL, and selects it.
4. **Connection is established.** The driver opens a socket connection to the PostgreSQL server, performs authentication, creates a session, and returns a `Connection` object.
5. **SQL execution.**
   ```java
   PreparedStatement ps = con.prepareStatement("SELECT * FROM employees");
   ResultSet rs = ps.executeQuery();
   ```
   SQL is converted into the PostgreSQL-specific protocol, sent to the database, executed, and the results come back.
6. **Result processing.**
   ```java
   while (rs.next()) {
       System.out.println(rs.getString("name"));
   }
   ```
   The driver converts DB data into Java types and populates the `ResultSet`.
7. **Resource cleanup.**
   ```java
   rs.close();
   ps.close();
   con.close();
   ```
   This frees DB connections and prevents memory leaks.

---

## 🔹 Statement vs PreparedStatement

**5. What is Statement?**
👉 Statement sends the full SQL query along with values to the database every time. The database has to read, compile, and run the query again and again. Because user input is directly added into the SQL, it is slow and unsafe, and it can lead to SQL injection.

**6. What is PreparedStatement?**
👉 PreparedStatement sends the SQL query only once with placeholders (`?`). The database compiles it once, and later only the values are sent separately. This makes it faster, reusable, and secure, because user input is treated as **data, not SQL code**, so SQL injection is prevented.

**7. What is the difference between Statement and PreparedStatement?**

| Statement | PreparedStatement |
|---|---|
| Full SQL sent every time | SQL sent once with `?` placeholders |
| Compiled again and again | Compiled once, reused |
| Slower | Faster |
| Unsafe (SQL injection) | Secure |

🔑 **One-line memory tip:**
Statement = simple but unsafe
PreparedStatement = fast and secure (recommended)

---

## 🔹 ResultSet, Errors, Code

**8. Demonstrate knowledge of executing and processing SQL queries using ResultSet.**
👉 ResultSet is used to process the data returned by a SQL SELECT query. After executing the query using Statement or PreparedStatement, the result is stored in a ResultSet. We use `next()` to move row by row and getter methods such as `getInt()`, `getString()`, or `getDouble()` to read column values.

```java
ResultSet rs = stmt.executeQuery("SELECT * FROM emp");
while (rs.next()) {
    System.out.println(rs.getInt("id") + " " + rs.getString("name"));
}
```
👉 In short: ResultSet acts like a **cursor** that reads database rows one by one.

**9. Discuss error handling and resource cleanup in JDBC. (Short)**
👉 Error handling is done using try-catch blocks to catch `SQLException`. Error details can be read with `getMessage()`, `getErrorCode()`, and `getSQLState()`.

Resource cleanup is important to avoid memory leaks. `ResultSet`, `Statement`, and `Connection` should always be closed after use. The best way is **try-with-resources**, which closes everything automatically, even if an exception occurs.

👉 In short: Handle exceptions with `SQLException` and always close JDBC resources.

**10. Write a short code snippet to fetch data from a database.**
```java
Connection con = DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/testdb",
        "user",
        "password");

PreparedStatement ps =
        con.prepareStatement("SELECT id, name FROM employee");

ResultSet rs = ps.executeQuery();

while (rs.next()) {
    System.out.println(rs.getInt("id") + " " + rs.getString("name"));
}

rs.close();
ps.close();
con.close();
```

**11. What is SQL Injection?**
👉 SQL injection is a security attack where a hacker inserts malicious SQL code into user input. If the application directly adds this input into an SQL query, the database may execute the attacker's code, allowing unauthorized access, data leakage, or data deletion.

---

## 🔹 Transaction & Batch

**12. What is a transaction in JDBC?**
👉 A transaction is a group of SQL queries executed as one unit of work. Its main purpose is to maintain data consistency and integrity.
- By default, a JDBC `Connection` has **auto-commit enabled**.
- Disable it with `setAutoCommit(false)`, run the queries, and call `con.commit()` if all succeed.
- If any operation fails, call `con.rollback()` to keep data consistent.

**13. What is JDBC Batch processing?**
👉 It executes multiple SQL operations together instead of sending each one separately. Use `addBatch()` to add operations and `executeBatch()` to run them. It improves performance by reducing database round trips. `PreparedStatement` with batch is commonly used for bulk insert and update.

**14. Show a JDBC batch insert example.**
```java
Connection con = DriverManager.getConnection(url, username, password);
con.setAutoCommit(false);

String sql = "INSERT INTO employee(id, name, salary) VALUES (?, ?, ?)";
PreparedStatement ps = con.prepareStatement(sql);

// Employee 1
ps.setInt(1, 101);
ps.setString(2, "Arun");
ps.setDouble(3, 50000);
ps.addBatch();

// Employee 2
ps.setInt(1, 102);
ps.setString(2, "Kumar");
ps.setDouble(3, 60000);
ps.addBatch();

// Employee 3
ps.setInt(1, 103);
ps.setString(2, "Ravi");
ps.setDouble(3, 55000);
ps.addBatch();

try {
    int[] result = ps.executeBatch();
    con.commit();
} catch (SQLException e) {
    con.rollback();
}
```

**15. What are the benefits of batch processing?**
👉 Fewer network round trips, faster bulk operations, and with auto-commit off the whole batch can commit or roll back as one unit.

---

# PART B: Extra Questions Expected in Interviews

## 🔹 Drivers & Connection

**16. What are the types of JDBC drivers?**

| Type | Name | Description |
|---|---|---|
| Type 1 | JDBC-ODBC bridge | Uses ODBC (removed from modern Java) |
| Type 2 | Native-API driver | Uses the DB's native client libraries |
| Type 3 | Network protocol driver | Talks to a middleware server |
| Type 4 | Thin / pure Java driver | Talks directly to the DB over the network. **Most used today** (PostgreSQL, MySQL) |

**17. Is `Class.forName()` still required?**
👉 No. Since JDBC 4.0 (Java 6), drivers are auto-discovered through the **Service Provider Interface** (`META-INF/services/java.sql.Driver`). Just having the JAR on the classpath is enough.

**18. What are the common JDBC URL formats?**
```
PostgreSQL : jdbc:postgresql://localhost:5432/testdb
MySQL      : jdbc:mysql://localhost:3306/schooldb
Oracle     : jdbc:oracle:thin:@localhost:1521:xe
SQL Server : jdbc:sqlserver://localhost:1433;databaseName=testdb
```

**19. What is DriverManager?**
👉 A class that manages the registered JDBC drivers. `DriverManager.getConnection(url, user, pass)` picks the driver matching the URL and returns a `Connection`.

**20. What is the difference between DriverManager and DataSource?**

| DriverManager | DataSource |
|---|---|
| Creates a new connection every time | Can provide pooled connections |
| URL and credentials hardcoded in code | Configured externally (JNDI / properties) |
| Basic, for small programs | Preferred for production |

**21. Why do we need connection pooling?**
👉 Creating a connection is expensive (socket, authentication, session). A pool (HikariCP, Apache DBCP, C3P0) keeps connections ready and reuses them, so the app is faster and the number of connections to the DB stays controlled.

**22. Is `Connection` thread-safe?**
👉 Not really. A `Connection` should not be shared between threads. Use one connection per thread/request (from a pool).

---

## 🔹 Statement Types & Execute Methods

**23. What are the types of statements in JDBC?**
👉
- `Statement` → static SQL
- `PreparedStatement` → precompiled SQL with parameters
- `CallableStatement` → stored procedures and functions

**24. What is the difference between `executeQuery()`, `executeUpdate()` and `execute()`?**

| Method | Used for | Returns |
|---|---|---|
| `executeQuery()` | SELECT | `ResultSet` |
| `executeUpdate()` | INSERT / UPDATE / DELETE / DDL | `int` (rows affected) |
| `execute()` | Any SQL (unknown type) | `boolean` (`true` if the result is a ResultSet) |

**25. Why is PreparedStatement faster than Statement?**
👉 The SQL is parsed and the execution plan is prepared once, then reused with different parameter values. Many databases and drivers also cache prepared statements.

**26. Can we use a placeholder (`?`) for a table or column name?**
👉 No. Placeholders work only for **values**. For dynamic table/column names, validate against a whitelist before building the SQL.

**27. How do you call a stored procedure in JDBC?**
```java
CallableStatement cs = con.prepareCall("{call get_employee(?, ?)}");
cs.setInt(1, 101);
cs.registerOutParameter(2, Types.VARCHAR);
cs.execute();
String name = cs.getString(2);
```

**28. How do you get the auto-generated key after an INSERT?**
```java
PreparedStatement ps = con.prepareStatement(
    "INSERT INTO employee(name, salary) VALUES (?, ?)",
    Statement.RETURN_GENERATED_KEYS);
ps.setString(1, "Arun");
ps.setDouble(2, 50000);
ps.executeUpdate();

ResultSet keys = ps.getGeneratedKeys();
if (keys.next()) {
    long id = keys.getLong(1);
}
```

---

## 🔹 ResultSet

**29. What is the initial position of the ResultSet cursor?**
👉 Before the first row. You must call `next()` once to move to the first row.

**30. What are the types of ResultSet?**

| Type | Meaning |
|---|---|
| `TYPE_FORWARD_ONLY` | Cursor moves forward only (default) |
| `TYPE_SCROLL_INSENSITIVE` | Scrollable, does not see DB changes made after the query |
| `TYPE_SCROLL_SENSITIVE` | Scrollable, reflects DB changes |

Concurrency: `CONCUR_READ_ONLY` (default) and `CONCUR_UPDATABLE`.

**31. How do you create a scrollable ResultSet?**
```java
Statement st = con.createStatement(
    ResultSet.TYPE_SCROLL_INSENSITIVE,
    ResultSet.CONCUR_READ_ONLY);
ResultSet rs = st.executeQuery("SELECT * FROM employee");
rs.last();      // jump to last row
rs.previous();  // move back
rs.absolute(3); // go to row 3
```

**32. What is the difference between `getString("name")` and `getString(1)`?**
👉 One reads by column **name**, the other by column **index** (starting at **1**, not 0). Name is more readable. Index is slightly faster.

**33. How do you handle NULL values from a ResultSet?**
👉 `getInt()` returns `0` for NULL. Use `rs.wasNull()` after reading, or `rs.getObject("col", Integer.class)` which returns `null`.

**34. What is ResultSetMetaData and DatabaseMetaData?**
👉
- `ResultSetMetaData` → info about the result's columns (count, names, types)
- `DatabaseMetaData` → info about the database (product name, version, tables, driver)

```java
ResultSetMetaData md = rs.getMetaData();
int columns = md.getColumnCount();
```

**35. What happens to the ResultSet when its Statement is closed?**
👉 It is closed automatically. Closing a `Connection` closes its statements and result sets too, but explicitly closing them is the best practice.

---

## 🔹 Error Handling & Resource Cleanup

**36. Show try-with-resources in JDBC.**
```java
String sql = "SELECT id, name FROM employee WHERE salary > ?";

try (Connection con = DriverManager.getConnection(url, user, pass);
     PreparedStatement ps = con.prepareStatement(sql)) {

    ps.setDouble(1, 40000);

    try (ResultSet rs = ps.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getInt("id") + " " + rs.getString("name"));
        }
    }
} catch (SQLException e) {
    System.out.println(e.getMessage() + " | " + e.getSQLState() + " | " + e.getErrorCode());
}
```
Resources are closed automatically in reverse order. No `finally` is needed.

**37. What information does SQLException give?**
👉 `getMessage()` (description), `getSQLState()` (standard 5-char code), `getErrorCode()` (vendor-specific code), `getNextException()` (chained exceptions).

**38. Is SQLException checked or unchecked?**
👉 **Checked.** You must catch it or declare `throws`. (Spring JDBC converts it into unchecked `DataAccessException`.)

**39. What are some common SQLException subclasses?**
👉 `SQLSyntaxErrorException`, `SQLIntegrityConstraintViolationException`, `SQLTimeoutException`, `BatchUpdateException`, `SQLTransientConnectionException`.

**40. What happens if you forget to close the connection?**
👉 A connection leak. The DB connections stay occupied, the pool runs out, and the application eventually fails with "connection not available" or "too many connections".

---

## 🔹 SQL Injection

**41. Show an example of SQL injection.**
```java
// ❌ Vulnerable
String sql = "SELECT * FROM users WHERE name = '" + name
           + "' AND password = '" + pass + "'";
// input: name = admin' --   → rest of the query becomes a comment
```

**42. How do you prevent SQL injection in JDBC?**
👉
- Use `PreparedStatement` with `?` placeholders
- Never concatenate user input into SQL
- Validate input, and whitelist dynamic identifiers (table/column/ORDER BY)
- Use least-privilege DB accounts
- Use stored procedures carefully (only with parameters)

```java
// ✅ Safe
PreparedStatement ps = con.prepareStatement(
    "SELECT * FROM users WHERE name = ? AND password = ?");
ps.setString(1, name);
ps.setString(2, pass);
```

---

## 🔹 Transactions (Advanced)

**43. What are ACID properties?**
👉
- **Atomicity** → all or nothing
- **Consistency** → data goes from one valid state to another
- **Isolation** → concurrent transactions don't interfere
- **Durability** → committed data survives failures

**44. Show a money-transfer transaction example.**
```java
try (Connection con = DriverManager.getConnection(url, user, pass)) {
    con.setAutoCommit(false);
    try (PreparedStatement debit = con.prepareStatement(
             "UPDATE account SET balance = balance - ? WHERE id = ?");
         PreparedStatement credit = con.prepareStatement(
             "UPDATE account SET balance = balance + ? WHERE id = ?")) {

        debit.setDouble(1, 5000);
        debit.setInt(2, 1);
        debit.executeUpdate();

        credit.setDouble(1, 5000);
        credit.setInt(2, 2);
        credit.executeUpdate();

        con.commit();
    } catch (SQLException e) {
        con.rollback();
        throw e;
    }
}
```

**45. What is a Savepoint?**
👉 A marker inside a transaction that lets you roll back **part** of the work instead of the whole transaction.

```java
Savepoint sp = con.setSavepoint("afterFirstInsert");
// ... more work
con.rollback(sp);   // undo only the work after the savepoint
con.commit();
```

**46. What are the transaction isolation levels in JDBC?**

| Constant | Problem prevented |
|---|---|
| `TRANSACTION_READ_UNCOMMITTED` | None (dirty reads allowed) |
| `TRANSACTION_READ_COMMITTED` | Dirty reads |
| `TRANSACTION_REPEATABLE_READ` | Dirty + non-repeatable reads |
| `TRANSACTION_SERIALIZABLE` | Dirty + non-repeatable + phantom reads |

```java
con.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
```

**47. What are dirty read, non-repeatable read and phantom read?**
👉
- **Dirty read** → reading uncommitted data from another transaction
- **Non-repeatable read** → the same row gives different values when read twice
- **Phantom read** → the same query returns a different set of rows when run twice

**48. What happens if you close a connection without commit or rollback when auto-commit is off?**
👉 It is driver-dependent. Some drivers commit and some roll back. Always call `commit()` or `rollback()` explicitly.

---

## 🔹 Batch (Advanced)

**49. What does `executeBatch()` return?**
👉 An `int[]` with the update count for each statement in the batch (or `Statement.SUCCESS_NO_INFO` if the driver can't give the count).

**50. What happens if one statement in a batch fails?**
👉 A `BatchUpdateException` is thrown. `getUpdateCounts()` shows which statements succeeded. Behavior after the failure (continue or stop) depends on the driver, so use a transaction and roll back to be safe.

**51. How do you batch a large number of records efficiently?**
👉 Execute in chunks (for example, every 500–1000 rows), then call `clearBatch()` to avoid memory build-up.

```java
int count = 0;
for (Employee e : employees) {
    ps.setInt(1, e.getId());
    ps.setString(2, e.getName());
    ps.setDouble(3, e.getSalary());
    ps.addBatch();

    if (++count % 1000 == 0) {
        ps.executeBatch();
        ps.clearBatch();
    }
}
ps.executeBatch();   // remaining records
con.commit();
```
For MySQL, add `rewriteBatchedStatements=true` to the URL. For PostgreSQL, `reWriteBatchedInserts=true`.

**52. Can Statement also do batch?**
👉 Yes. `Statement.addBatch(sql)` works with different SQL strings. `PreparedStatement` batch is for the same SQL with different parameters, and it is the one commonly used.

---

## 🔹 Misc & Scenario Questions

**53. What is the difference between `java.util.Date` and `java.sql.Date`?**
👉 `java.sql.Date` extends `java.util.Date` and holds only the date (no time) for SQL `DATE`. Related: `java.sql.Time`, `java.sql.Timestamp`. With Java 8+, prefer `LocalDate` / `LocalDateTime` via `ps.setObject(1, LocalDate.now())`.

**54. How do you store and read large data (BLOB / CLOB)?**
👉 Use `setBinaryStream()` / `getBinaryStream()` for BLOB (images, files) and `setCharacterStream()` / `getClob()` for CLOB (large text).

**55. How do you set a query timeout and fetch size?**
```java
ps.setQueryTimeout(10);   // seconds
ps.setFetchSize(500);     // rows fetched per round trip
```

**56. What are the limitations of plain JDBC?**
👉
- Lots of boilerplate (connections, try-catch-finally)
- Manual mapping of `ResultSet` to objects
- Checked `SQLException` everywhere
- No built-in pooling, caching, or relationship handling
- Easy to leak resources

**57. How does Spring JDBC improve on plain JDBC?**
👉 `JdbcTemplate` manages connections and resources, maps rows through `RowMapper`, and converts `SQLException` into unchecked `DataAccessException`, while you still write the SQL.

**58. JDBC vs Hibernate/JPA?**

| JDBC | Hibernate / JPA |
|---|---|
| Write raw SQL | Object-based (HQL/JPQL, auto SQL) |
| Manual mapping | Automatic ORM mapping |
| Full SQL control and speed | Less control, more productivity |
| No caching | First/second-level cache |

**59. Scenario: your app throws "Too many connections". What do you check?**
👉 Connection leaks (not closed), missing pooling, oversized pool, long-running queries or transactions holding connections, and the DB's `max_connections` limit.

**60. Scenario: how do you read a very large table without running out of memory?**
👉 Use `setFetchSize()` (and for PostgreSQL turn auto-commit off so it uses a cursor), process rows one at a time inside the `while (rs.next())` loop, and never load everything into a list.

**61. Scenario: how do you insert records with an ID from a sequence/auto-increment and use it in a child table?**
👉 Insert the parent with `RETURN_GENERATED_KEYS`, read the key with `getGeneratedKeys()`, then insert the child rows using that key, all in one transaction.
