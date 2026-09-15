# 21 — JSON Queries

> **Priority:** Advanced  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Extract, modify, aggregate, tabularize, and index JSON while distinguishing SQL null from JSON null.

## Syntax template

```sql
SELECT doc->'$.profile.name' AS json_name,
       doc->>'$.profile.name' AS text_name
FROM events;

SELECT e.event_id, jt.*
FROM events AS e
JOIN JSON_TABLE(e.doc, '$.items[*]' COLUMNS (
  product_id BIGINT PATH '$.id',
  qty INT PATH '$.qty' NULL ON EMPTY
) ) AS jt ON TRUE;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `JSON_EXTRACT` / `->` | JSON value. | Missing path → SQL `NULL`; JSON `null` is a JSON value. |
| `->>` | Unquoted scalar text representation. | Missing path → SQL `NULL`; JSON null handling differs from missing path in type tests. |
| `JSON_SET` | Document with path inserted/replaced. | SQL-null required argument can make result `NULL`; JSON null must be constructed/stored deliberately. |
| `JSON_CONTAINS` | `1`/`0` containment result. | Null required argument → `NULL`. |
| `JSON_ARRAYAGG` | JSON array per group. | Can include JSON null elements for SQL-null values; no rows → `NULL`. |
| `JSON_TABLE` | Relational rows/columns. | `NULL/DEFAULT/ERROR ON EMPTY|ERROR` controls missing/error behavior. |

## Revision notes

- SQL `NULL` means absence at the SQL layer; JSON `null` is a value inside a valid JSON document.
- Use `JSON_TYPE(JSON_EXTRACT(doc,path))` plus `IS NULL` to distinguish types and missing paths.
- Validate paths and decide explicit `ON EMPTY`/`ON ERROR` behavior for `JSON_TABLE` columns.
- Index frequently queried scalar paths using generated columns or supported functional/multi-valued index patterns.
- Do not use JSON as a substitute for stable relational columns that require joins, constraints, and common filtering.

## Examples

### Missing path

```sql
SELECT JSON_EXTRACT('{"a":1}', '$.b') IS NULL AS missing;
```

**Expected behavior:** Returns `1` because the path is absent and extraction returns SQL null.

### Modify without replacing document

```sql
SELECT JSON_SET('{"a":1}', '$.b', 2);
```

**Expected behavior:** Returns `{"a": 1, "b": 2}` as JSON.

## Tricky parts

- Confusing missing path, SQL null, JSON null, and string `"null"`.
- Using `->` when unquoted text is needed.
- Ignoring `ON EMPTY`/`ON ERROR`.
- Storing heavily relational data only in JSON.
- Filtering JSON paths repeatedly without an index strategy.

## Interview checks

1. SQL null versus JSON null?
2. `->` versus `->>`?
3. How do you index a commonly queried JSON property?

## 30-second recap

- Extract, modify, aggregate, tabularize, and index JSON while distinguishing SQL null from JSON null.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — JSON Queries](https://dev.mysql.com/doc/refman/8.4/en/json-functions.html)
