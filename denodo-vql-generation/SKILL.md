---
name: denodo-vql-generation
description: Guidelines to follow when generating valid Denodo VQL queries to execute in the Denodo Platform.
license: Complete terms in LICENSE.txt
---

# VQL Generation

1. View names must use the format "<database_name>"."<view_name>". For instance,
if the database is 'organization' and the view is 'employees', reference it as: "organization"."employees".

2. Column names and column aliases must be wrapped in double quotes.

Wrong (unquoted view name without database):

```vql
SELECT * FROM employees;
```

Correct (quoted database and view, quoted columns and aliases):

```vql
SELECT
    "c"."CustomerID" AS "Customer",
    "o"."OrderID"
FROM
    "お客様"."Clients" c
JOIN
    "お客様"."Orders" o
ON
    "c"."CustomerID" = "o"."CustomerID";
```

3. CAST is supported in VQL with these types:

- BOOL
- CHAR
- DECIMAL
- FLOAT
- FLOAT4
- FLOAT8
- INT2
- INT4
- INT8
- INTEGER
- REAL
- TEXT
- VARCHAR

Valid example using INT2:

```vql
SELECT CAST("quantity" AS INT2) FROM "organization"."hardware_bv"
```

4. VQL protected words cannot be used as aliases. The list of protected words is:

- BASE
- DF
- NOS
- OBL
- WS

5. VQL strings use single quotes: WHERE name = 'Joseph'. Single quotes can be escaped in VQL strings by doubling them: 'D''angelo' matches "D'angelo"

6. VQL string functions

- **SUBSTR**(`string`, `start index`, `length?`). Index starts at 1. The length sets the number of characters to return and includes the index character. If `length` is omitted, the substring returns everything until the end of the string.
  - SUBSTR('Artificial', 2, 5) → 'rtifi'
  - SUBSTR('Artificial', 2) → 'rtificial'

- **CONCAT**(`str1`, `str2`, ..., `strN`). Joins strings together. Requires 2+ parameters.
  - CONCAT(GETYEAR(date), '-', GETMONTH(date), '-', GETDAY(date))

- **LEN**(`string`). Returns string length.

- **POSITION**(`needle IN haystack`). Finds starting position of a substring.
  - POSITION('no' IN 'Denodo') → 3

7. Subqueries (a CTE is also considered a subquery) are not allowed in HAVING clauses in VQL. Only simple conditions or aggregate functions comparing results to static values or other aggregates are permitted.

Valid example of a HAVING clause in VQL:

```vql
SELECT "department", AVG(salary) AS "avg_salary"
FROM "organization"."employees"
GROUP BY "department"
HAVING AVG(salary) > 30000;
```

Example of invalid HAVING clause (uses subquery `SELECT department...`):

```vql
SELECT "department", AVG(salary) AS "avg_salary"
FROM "organization"."employees"
GROUP BY "department"
HAVING "department" IN (SELECT "department" FROM "organization"."managers" WHERE "status" = 'senior');
```

Example of invalid VQL HAVING clause because it uses a CTE:

```vql
WITH salary_threshold AS (
    SELECT 30000 AS threshold
)
SELECT "department", AVG(salary) AS "avg_salary"
FROM "organization"."employees", salary_threshold
GROUP BY "department"
HAVING AVG(salary) > threshold;
```

8. In VQL, LIMIT and FETCH can only be can only be used in the outer query and inside a CTE/FROM/WHERE clause subquery. LIMIT and FETCH cannot be used inside a SELECT clause subquery.

This query works, because LIMIT is in the outer query:

```vql
SELECT *
FROM "bank"."customers"
LIMIT 3;
```

This query fails, because it uses LIMIT in a SELECT subquery:

```vql
SELECT
    "c"."customer_id",
    (
        SELECT "customer_id"
        FROM "bank"."loans"
        GROUP BY "customer_id"
        ORDER BY SUM("loan_amount") DESC
        LIMIT 1
    ) AS "top_loan_customer"
FROM "bank"."customers" c;
```

Some versions of Denodo don't accept LIMIT or FETCH in ANY subquery. If you face this error, review [how to fix LIMIT in subquery](references/fix_limit_in_subquery.md)

9. In VQL, NULLS LAST is invalid in ORDER BY.

10. In VQL, you cannot use aggregate functions directly in the ORDER BY clause if they are not projected in the SELECT list or projected but inside another function. To avoid this:

- Create the alias name for the projected field
- Use alias name in the ORDER BY clause.

Valid example of using an aggregate function in an ORDER BY clause:

```vql
SELECT "user", ROUND(SUM("amount") / 100000000, 2) AS "total_amount"
FROM "bank"."customer"
GROUP BY "user"
ORDER BY "total_amount";
```

Invalid example of using an aggregate function in an ORDER BY clause:

```vql
SELECT "user", ROUND(SUM("amount") / 100000000, 2) AS "total_amount"
FROM "bank"."customer"
GROUP BY "user"
ORDER BY SUM("amount");
```

## Extra references

This skill contains generic VQL generation guidelines. When working with any of the following specific scenarios, you must also read the appropiate reference file to understand how to work with that scenario:

- [Arithmetic](references/arithmetic.md)
- [Aggregation](references/aggregation.md)
- [Casting](references/casting.md)
- [Dates](references/dates.md)
- [Text](references/text.md)
- [JSON](references/json.md)
- [XML](references/xml.md)
- [Vector search](references/vector-search.md). Use this when working with views that contains vectors.
- [Metrics](references/metrics.md). Use this when working with metric views.
- [LLM functions](references/llm-functions.md). Advanced LLM-powered functionality for summarization, translation, etc. Can only be executed with the specific consent of the user.
- [Spatial](references/spatial.md)
- [Window functions](references/window-functions.md)
