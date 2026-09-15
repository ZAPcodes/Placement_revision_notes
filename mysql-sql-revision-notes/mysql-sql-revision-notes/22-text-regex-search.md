# 22 — Text Search and Regular Expressions

> **Priority:** Advanced  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Use regex and full-text search with correct return semantics and performance expectations.

## Syntax template

```sql
SELECT REGEXP_LIKE(text_col, '^[A-Z][a-z]+$');
SELECT REGEXP_REPLACE(text_col, '[[:space:]]+', ' ');

SELECT id, MATCH(title, body) AGAINST (? IN BOOLEAN MODE) AS score
FROM articles
WHERE MATCH(title, body) AGAINST (? IN BOOLEAN MODE);
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `REGEXP_LIKE` | `1` if matched, else `0`. | Null expression/pattern → `NULL`; invalid pattern → error. |
| `REGEXP_INSTR` | 1-based match position; `0` when not found. | Null required input → `NULL`. |
| `REGEXP_SUBSTR` | Matched substring. | No match or null input → `NULL`. |
| `REGEXP_REPLACE` | String with matches replaced. | Null required input → `NULL`. |
| `MATCH ... AGAINST` | Relevance score. | Rows with null text contribute no terms; nonmatches score `0`. |

## Revision notes

- Regex syntax is ICU-based in modern MySQL; collation and match-type flags affect case sensitivity.
- Use full-text search for word relevance and boolean term search; use regex for structural patterns.
- A full-text index must cover the same column list used by `MATCH` in typical indexed use.
- Stopwords, minimum token size, parser, and language affect matches.
- Regex predicates usually cannot use a normal B-tree index for arbitrary patterns.

## Examples

### Find malformed emails conceptually

```sql
SELECT email
FROM customers
WHERE NOT REGEXP_LIKE(email, '^[^@]+@[^@]+\.[^@]+$')
   OR email IS NULL;
```

**Expected behavior:** Returns null emails explicitly plus strings not matching the simplified pattern.

### Boolean full-text

```sql
SELECT article_id
FROM articles
WHERE MATCH(title, body)
      AGAINST('+mysql -oracle' IN BOOLEAN MODE);
```

**Expected behavior:** Articles required to contain `mysql` and exclude `oracle`, subject to full-text rules.

## Tricky parts

- Using a toy regex as complete email validation.
- Forgetting null rows need explicit handling with `OR ... IS NULL`.
- Expecting regex to be index-fast.
- Ignoring stopwords/token-size settings.
- Calling `MATCH` with a different column list than the intended index.

## Interview checks

1. No-match result for `REGEXP_INSTR` versus `REGEXP_SUBSTR`?
2. Regex versus full-text search?
3. What can make an expected word absent from full-text results?

## 30-second recap

- Use regex and full-text search with correct return semantics and performance expectations.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Text Search and Regular Expressions](https://dev.mysql.com/doc/refman/8.4/en/regexp.html)
