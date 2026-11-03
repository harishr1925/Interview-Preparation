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

---

# PART C: Advanced Questions Expected in Interviews

## 🔹 Connection Pool & Connection Internals

**62. What happens when you call `close()` on a pooled connection?**
👉 The pool hands you a **proxy (wrapper)** around the real connection. `close()` on the proxy does **not** close the physical socket. It resets the connection state (auto-commit, read-only, isolation, and rolls back any uncommitted work) and **returns it to the pool** for reuse. This is why you must still call `close()` (or use try-with-resources): that is how the connection goes back.

**63. How does a pool deal with stale or broken connections (for example after a DB restart or a firewall timeout)?**
👉
- The pool **validates** a connection before handing it out (`Connection.isValid()` in JDBC 4, or a test query such as `SELECT 1`)
- It **retires old connections** (`maxLifetime`) before the DB or firewall silently kills them. Keep `maxLifetime` **shorter** than the DB/firewall idle timeout (for example MySQL `wait_timeout`)
- `keepaliveTime` pings idle connections
- Broken connections are evicted and replaced

**64. What are the different timeouts in JDBC?**

| Timeout | Meaning |
|---|---|
| Login / connect timeout | Max time to **establish** a connection to the DB |
| Pool connection timeout | Max time a thread **waits for a free connection** from the pool (HikariCP default 30s) |
| Socket (read) timeout | Max time to wait for **data from the network** |
| Query timeout | `Statement.setQueryTimeout(sec)`: max time a **single statement** may run before the driver cancels it |

Set all of them. A missing socket/query timeout is a common reason for hung threads.

**65. Why is HikariCP considered fast?**
👉 A small, optimized codebase, a lock-light `ConcurrentBag` for borrowing connections with **thread-local caching**, minimal overhead proxies, and sensible defaults. It is the default pool in Spring Boot.

**66. How do you find connection leaks?**
👉
- Enable the pool's leak detection (HikariCP `leakDetectionThreshold`, for example 20000 ms). It logs a **stack trace** of the code that took the connection and didn't return it
- Watch the pool metrics (active vs idle vs waiting threads)
- On the DB side, check active sessions (for example PostgreSQL `pg_stat_activity`)
- Fix by always using try-with-resources

---

## 🔹 Statements & ResultSets (Advanced)

**67. How does PreparedStatement caching work?**
👉 Preparing SQL has a cost, so drivers cache it.
- **PostgreSQL JDBC:** switches to a **server-side prepared statement** after the same statement runs `prepareThreshold` times (default 5)
- **MySQL Connector/J:** enable `cachePrepStmts=true`, `prepStmtCacheSize`, `prepStmtCacheSqlLimit` (and optionally `useServerPrepStmts`)

Building SQL by **string concatenation** produces a different SQL text every time, which defeats caching and also invites SQL injection.

**68. Why does `setFetchSize()` sometimes not work (for example in PostgreSQL)?**
👉 PostgreSQL's driver loads the **whole result set into memory** unless all of these are true: auto-commit is **off**, the ResultSet is `TYPE_FORWARD_ONLY`, and it is a single statement. Then it uses a cursor and fetches in chunks of the fetch size.
MySQL loads everything by default. Use `setFetchSize(Integer.MIN_VALUE)` (streaming) or `useCursorFetch=true`.

**69. What is an updatable ResultSet?**
👉 A ResultSet created with `CONCUR_UPDATABLE` lets you modify rows through the ResultSet itself.

```java
Statement st = con.createStatement(ResultSet.TYPE_SCROLL_INSENSITIVE, ResultSet.CONCUR_UPDATABLE);
ResultSet rs = st.executeQuery("SELECT id, name FROM employee");
rs.absolute(2);
rs.updateString("name", "Kumar");
rs.updateRow();            // UPDATE
rs.moveToInsertRow();
rs.updateString("name", "Ravi");
rs.insertRow();            // INSERT
```
It is rarely used in real projects (limited to simple single-table queries, driver-dependent). Explicit SQL is preferred.

**70. What is a RowSet / CachedRowSet?**
👉 `RowSet` is a JavaBean-style wrapper over a ResultSet. `CachedRowSet` is a **disconnected** RowSet: it loads the data, closes the connection, and you can still read (and even modify and sync back) the data. It is serializable. It is rarely used today (DTO lists are more common).

**71. How do you handle multiple result sets from one call (stored procedure)?**
```java
boolean isResultSet = cs.execute();
while (true) {
    if (isResultSet) {
        try (ResultSet rs = cs.getResultSet()) { /* read rows */ }
    } else if (cs.getUpdateCount() == -1) {
        break;                       // no more results
    }
    isResultSet = cs.getMoreResults();
}
```

**72. How do you insert a NULL value with PreparedStatement?**
👉 Use `ps.setNull(index, Types.VARCHAR)` (or `setObject(index, null, Types.VARCHAR)`). Plain `setObject(index, null)` can fail on some databases because the driver can't infer the type.

**73. How do you use an IN clause with PreparedStatement?**
👉 `?` can't hold a list. Build the placeholders dynamically:

```java
String placeholders = String.join(",", Collections.nCopies(ids.size(), "?"));
String sql = "SELECT * FROM employee WHERE id IN (" + placeholders + ")";
try (PreparedStatement ps = con.prepareStatement(sql)) {
    int i = 1;
    for (Long id : ids) ps.setLong(i++, id);
    ResultSet rs = ps.executeQuery();
}
```
Only the `?` are concatenated, never user values. Note: Oracle allows a maximum of 1000 items in an IN list. PostgreSQL also supports `= ANY(?)` with an array.

**74. Why use `BigDecimal` and not `double` for money?**
👉 `double` has binary floating-point errors (`0.1 + 0.2 = 0.30000000000000004`). Use `NUMERIC/DECIMAL` columns with `rs.getBigDecimal()` / `ps.setBigDecimal()`.

**75. How do you handle dates, time and time zones correctly in JDBC?**
👉
- `java.sql.Timestamp` uses the **JVM default time zone**, which causes shifts (for example, a 5:30 hour difference) when the app and DB zones differ
- Prefer storing in **UTC**, use `TIMESTAMP WITH TIME ZONE` where available
- JDBC 4.2 supports `ps.setObject(1, LocalDateTime.now())` and `OffsetDateTime`
- Set driver time zone properties where needed (for example MySQL `connectionTimeZone` / `serverTimezone`)

**76. Are Statement and ResultSet thread-safe? Can one Statement have many open ResultSets?**
👉 No, don't share them between threads. Also, **re-executing a Statement closes its previous ResultSet**, so you get one active ResultSet per Statement. For nested loops (read parent rows, query children), use a separate statement.

---

## 🔹 Transactions & Concurrency (Advanced)

**77. How do you lock a row in plain JDBC (`SELECT ... FOR UPDATE`)?**
```java
con.setAutoCommit(false);
try (PreparedStatement ps = con.prepareStatement(
        "SELECT balance FROM account WHERE id = ? FOR UPDATE")) {
    ps.setInt(1, 1);
    ResultSet rs = ps.executeQuery();    // row is now locked
    rs.next();
    double balance = rs.getDouble(1);
    // check and update, then:
    con.commit();                        // lock released
}
```
Other transactions that touch the row wait until commit/rollback. Keep this transaction short.

**78. How do you implement optimistic locking in plain JDBC?**
👉 Add a `version` column and check it in the UPDATE:

```java
int rows = ps.executeUpdate();   // UPDATE student SET name=?, version=version+1 WHERE id=? AND version=?
if (rows == 0) {
    throw new IllegalStateException("Record changed by another user");
}
```

**79. What is a distributed transaction (XA / JTA / two-phase commit)?**
👉 A transaction spanning **multiple resources** (two databases, or a DB and a message broker). The coordinator runs **2PC**: phase 1 *prepare* (all resources vote), phase 2 *commit* or *rollback*. JDBC supports it through `XADataSource` plus a JTA manager (Atomikos, Narayana).
Drawbacks: slow, locks held longer, coordinator failure complexity. Modern systems often prefer **eventual consistency** (Saga pattern, outbox pattern).

**80. How do you handle deadlocks and transient failures in JDBC?**
👉 Catch the `SQLException`, check `getSQLState()` (for example `40001` serialization failure / MySQL deadlock, `40P01` PostgreSQL deadlock), **roll back, and retry the whole transaction** with a small backoff and a maximum number of attempts. Only retry if the operation is safe to repeat (idempotent). Prevention: short transactions, consistent lock order.

**81. How do you do an "insert or update" (upsert) in JDBC?**
👉 Use the database's native upsert, atomic and safe under concurrency:

```sql
-- PostgreSQL
INSERT INTO employee(id, name) VALUES (?, ?)
ON CONFLICT (id) DO UPDATE SET name = EXCLUDED.name;

-- MySQL
INSERT INTO employee(id, name) VALUES (?, ?)
ON DUPLICATE KEY UPDATE name = VALUES(name);

-- Oracle / SQL Server: MERGE
```
Avoid "SELECT then INSERT or UPDATE", which has a race condition.

---

## 🔹 Security, Design & Tuning

**82. What is second-order SQL injection? And what about LIKE wildcards?**
👉
- **Second-order injection:** malicious input is stored safely first, but later **read from the DB and concatenated** into another query. Treat DB data as untrusted too, and always use parameters.
- **LIKE wildcards:** parameters stop SQL injection but `%` and `_` typed by a user still act as wildcards. Escape them (`ESCAPE '\'`) when you want a literal match.

**83. What are the JDBC security best practices?**
👉
- Always use `PreparedStatement` / parameters
- Don't hardcode credentials. Use env variables or a secrets vault
- Use a **least-privilege** DB user (no DDL/admin rights for the app)
- Use TLS/SSL for the DB connection
- Don't log passwords or sensitive parameter values
- Validate input and whitelist dynamic identifiers

**84. How would you design a clean DAO class using plain JDBC?**
👉
- Inject a `DataSource` (pooled), don't use a static shared `Connection`
- Get a connection per operation (or per transaction) and use try-with-resources
- Keep SQL as constants, map rows in one place
- Let the service layer control transactions (pass the `Connection`, or use a transaction helper)

```java
public class EmployeeDao {
    private final DataSource ds;
    public EmployeeDao(DataSource ds) { this.ds = ds; }

    public Optional<Employee> findById(int id) throws SQLException {
        String sql = "SELECT id, name FROM employee WHERE id = ?";
        try (Connection con = ds.getConnection();
             PreparedStatement ps = con.prepareStatement(sql)) {
            ps.setInt(1, id);
            try (ResultSet rs = ps.executeQuery()) {
                return rs.next()
                    ? Optional.of(new Employee(rs.getInt("id"), rs.getString("name")))
                    : Optional.empty();
            }
        }
    }
}
```

**85. How do you log the SQL and parameters in plain JDBC?**
👉 Use a JDBC proxy/logging tool such as **p6spy** or **datasource-proxy**, or the driver's own logging (for example PostgreSQL `loggerLevel`). Use it in dev/test only, and avoid logging sensitive values.

**86. What are the main JDBC performance tuning tips?**
👉
- Use a connection pool and size it sensibly
- Use `PreparedStatement` (with caching) and batching for bulk work
- Select only the columns you need (no `SELECT *`)
- Set a suitable fetch size for large reads
- Keep transactions short, and avoid remote calls inside them
- Add proper **indexes** and check the query plan (`EXPLAIN`)
- Avoid the "N+1 queries in a Java loop" pattern. Use a JOIN or an IN query

---

## 🔹 Scenario Questions (Advanced)

**87. Scenario: two users transfer money from the same account at the same time. How do you prevent a wrong balance?**
👉 Run it in one transaction and make the check and the update **atomic**:

```sql
UPDATE account SET balance = balance - ? WHERE id = ? AND balance >= ?
```
If `executeUpdate()` returns 0, the balance was insufficient, so roll back. Or lock rows with `SELECT ... FOR UPDATE`. To avoid deadlocks between two transfers (A→B and B→A), always lock the accounts in a **consistent order** (for example, lowest id first).

**88. Scenario: after the database restarted (or failed over), the app keeps throwing "connection reset / closed" errors until you restart the app. Why?**
👉 The pool is holding **stale connections** that the DB already dropped. Fix: enable validation (`isValid` / test query), set `maxLifetime` below the DB/firewall idle timeout, enable `keepaliveTime`, and make sure retry logic exists for transient failures.

**89. Scenario: under load the application freezes with no errors. How do you investigate?**
👉
1. Take **thread dumps**: many threads waiting in `getConnection()` means pool exhaustion
2. Check for connection leaks (leak detection) and long transactions/slow queries holding connections
3. Check for missing **socket/query timeouts** (threads stuck on a DB call)
4. Check for remote calls inside transactions
5. Check DB side for locks/blocked sessions

**90. Scenario: one bad row in a batch of 10,000 fails the whole batch. How do you handle it?**
👉 Catch `BatchUpdateException` and use `getUpdateCounts()` to find which rows succeeded or failed. Options: roll back and **re-run the failed chunk row by row** to isolate and log the bad record, or validate data before inserting. Use smaller chunks (for example 500) so a failure affects less data.

**91. Scenario: you must export 1 crore (10 million) rows to a CSV file. How do you do it without running out of memory?**
👉 Use a forward-only, read-only ResultSet with a fetch size (auto-commit off for PostgreSQL), and **write each row to the file as you read it** with a buffered writer. Select only needed columns and never build a full list in memory.

**92. Scenario: a search screen has 6 optional filters. How do you build the SQL safely?**
👉 Build the SQL dynamically from **fixed fragments** and collect the values in a list, then bind them as parameters:

```java
StringBuilder sql = new StringBuilder("SELECT * FROM employee WHERE 1=1");
List<Object> params = new ArrayList<>();

if (name != null)   { sql.append(" AND name LIKE ?");   params.add("%" + name + "%"); }
if (minSalary != null) { sql.append(" AND salary >= ?"); params.add(minSalary); }

try (PreparedStatement ps = con.prepareStatement(sql.toString())) {
    for (int i = 0; i < params.size(); i++) ps.setObject(i + 1, params.get(i));
    ...
}
```
Only fixed SQL text is appended, and user input only goes through `?`. For dynamic sort columns use a whitelist.

**93. Scenario: legacy code builds SQL with `Statement` and string concatenation. How do you migrate it safely?**
👉
- Find all concatenated queries and replace user-supplied values with `?` + `PreparedStatement`
- Whitelist any dynamic table/column/sort names
- Add tests for the key queries before and after
- Use code scanning (SAST) to catch the remaining ones
- Make sure the DB account is least-privilege as a safety net

**94. Scenario: an audit log must be saved even if the main transaction rolls back. How?**
👉 Write the audit record using a **separate connection** (its own transaction) and commit it independently of the main business transaction. In Spring this is `REQUIRES_NEW`. In plain JDBC, simply take a second connection from the pool.

**95. Scenario: timestamps are 5 hours 30 minutes off between the app and the DB. What do you check?**
👉 A time zone mismatch: JVM default zone vs DB session zone vs column type (`TIMESTAMP` vs `TIMESTAMPTZ`). Store UTC, use `OffsetDateTime`/`TIMESTAMP WITH TIME ZONE`, check driver properties (for example MySQL `connectionTimeZone`), and make the zone explicit instead of relying on defaults.

**96. Scenario: two requests try to insert the same record at the same time, and one fails with a duplicate key error. What is the right fix?**
👉 Put a **unique constraint** on the DB (the real guard) and use an atomic upsert (`ON CONFLICT` / `ON DUPLICATE KEY` / `MERGE`) or catch the duplicate-key `SQLException` (`SQLState 23505` in PostgreSQL, error 1062 in MySQL) and handle it. "Check if it exists, then insert" in the application is **not safe** under concurrency.

**97. Scenario: the query is fast in the SQL client but slow from the Java app. What could be wrong?**
👉
- **Parameter type mismatch** (for example `setString()` on a numeric/indexed column causes an implicit cast, so the index is not used)
- Different plan for the prepared statement vs literal values
- The query runs in a **Java loop** (N+1), so network round trips add up
- Large fetch with default fetch size, or the whole result loaded into memory
- Waiting on the pool (`getConnection()` time) rather than on the query itself

Compare the real executed SQL, check the plan with `EXPLAIN`, and measure where the time is spent.

**98. Scenario: a deadlock error appears in production now and then. What do you do?**
👉 Read the DB's deadlock report to see which statements and rows conflicted. Then:
- Access tables/rows in a **consistent order** everywhere
- Keep transactions **short** (no remote calls or user waiting inside)
- Add the missing index (a full scan locks more rows)
- Add **retry** logic for the deadlock error (SQLState `40001` / `40P01`)
