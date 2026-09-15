# MySQL SQL Revision Notes

Placement-focused SQL notes using **MySQL 8.4 LTS syntax**. These are revision sheets, not a database textbook.

Every chapter contains:

- a syntax template;
- exact keyword/return behavior;
- explicit `NULL` behavior;
- compact examples with expected results;
- common traps and interview checks.

## Recommended order

1. First pass: `01–08`
2. Query-building pass: `09–14`
3. Schema and concurrency: `15–17`
4. Interview priority: `20`, `24`, `25`
5. MySQL extras: `18`, `19`, `21–23`
6. Final revision: `99-FINAL-CHEATSHEET.md`

## Shared example vocabulary

Examples assume conventional tables such as:

- `departments(department_id, name)`
- `employees(employee_id, department_id, manager_id, first_name, last_name, salary, active)`
- `customers(customer_id, email)`
- `orders(order_id, customer_id, ordered_at, amount, status)`
- `products(product_id, stock, price)`
- `order_items(order_id, product_id, qty, unit_price)`

Adapt identifiers to the schema in the question.

## Chapter index

| File | Topic | Priority |
|---|---|---|
| [01-mysql-fundamentals.md](01-mysql-fundamentals.md) | SQL and MySQL Fundamentals | Core |
| [02-select-queries.md](02-select-queries.md) | Basic SELECT Queries | Core |
| [03-where-filtering.md](03-where-filtering.md) | Filtering with WHERE | Core |
| [04-sorting-limiting-pagination.md](04-sorting-limiting-pagination.md) | Sorting, Limiting, and Pagination | Core |
| [05-expressions-null-conditional.md](05-expressions-null-conditional.md) | Expressions, Conditional Logic, and NULL | Core |
| [06-built-in-functions.md](06-built-in-functions.md) | Built-in MySQL Functions | Core |
| [07-aggregation-grouping.md](07-aggregation-grouping.md) | Aggregate Functions and Grouping | Core |
| [08-joins.md](08-joins.md) | Joins | Core |
| [09-set-operations.md](09-set-operations.md) | Set Operations | Core |
| [10-subqueries.md](10-subqueries.md) | Subqueries | Core |
| [11-ctes.md](11-ctes.md) | Common Table Expressions | Core |
| [12-window-functions.md](12-window-functions.md) | Window Functions | Core |
| [13-insert-data.md](13-insert-data.md) | Inserting Data | Core |
| [14-update-delete.md](14-update-delete.md) | Updating and Deleting Data | Core |
| [15-ddl-tables.md](15-ddl-tables.md) | Database and Table Definition Queries | Reference |
| [16-constraints-indexes.md](16-constraints-indexes.md) | Keys, Constraints, and Index Queries | Core |
| [17-transactions-locking.md](17-transactions-locking.md) | Transactions and Concurrency Queries | Core |
| [18-views-temp-results.md](18-views-temp-results.md) | Views and Temporary Results | Reference |
| [19-stored-objects.md](19-stored-objects.md) | Stored SQL Objects | Reference |
| [20-query-optimization.md](20-query-optimization.md) | Query Performance and Optimization | Core |
| [21-json-queries.md](21-json-queries.md) | JSON Queries | Advanced |
| [22-text-regex-search.md](22-text-regex-search.md) | Text Search and Regular Expressions | Advanced |
| [23-users-privileges-security.md](23-users-privileges-security.md) | Users, Privileges, and Security Queries | Reference |
| [24-interview-query-patterns.md](24-interview-query-patterns.md) | Essential Interview Query Patterns | Core |
| [25-mysql-traps-differences.md](25-mysql-traps-differences.md) | MySQL Traps and Dialect Differences | Core |
| [99-FINAL-CHEATSHEET.md](99-FINAL-CHEATSHEET.md) | Foundational syntax, dates, windows, and null rules | Final revision |

## Version notes

- Target: MySQL 8.4 LTS.
- `INTERSECT` and `EXCEPT` are available in modern MySQL but not older 8.0 releases before 8.0.31.
- MySQL has no direct `FULL OUTER JOIN`, `QUALIFY`, or ordinary materialized-view statement.
- Behavior can depend on `sql_mode`, collation, storage engine, and session time zone; verify these in real systems.

Official starting points: [SELECT](https://dev.mysql.com/doc/refman/8.4/en/select.html), [NULL values](https://dev.mysql.com/doc/refman/8.4/en/working-with-null.html), [date functions](https://dev.mysql.com/doc/refman/8.4/en/date-and-time-functions.html), and [window functions](https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html).
