---
title: "QUALIFY in SQL"
description: "Using the QUALIFY keyword in SQL."
date: "2025-04-22"
---

Last year, while starting to use Snowflake, I discovered the `QUALIFY` keyword in SQL.

`QUALIFY` filters results based on a window function. It's not part of the SQL standard and so won't be supported in all SQL engines, but I think it has good coverage in some of the newer ones. 

Let's say we have an `employees` table, courtesy of ChatGPT:

```sql
┌────┬─────────┬─────────────┬────────┐
│ id ┆ name    ┆ department  ┆ salary │
╞════╪═════════╪═════════════╪════════╡
│  1 ┆ Alice   ┆ Engineering ┆  70000 │
│  2 ┆ Bob     ┆ Engineering ┆  72000 │
│  3 ┆ Charlie ┆ HR          ┆  65000 │
│  4 ┆ Diana   ┆ HR          ┆  68000 │
│  5 ┆ Eve     ┆ Engineering ┆  71000 │
└────┴─────────┴─────────────┴────────┘
```

We want to find the top earner for each department. With `QUALIFY`, the query becomes concise:

```sql
SELECT * 
FROM employees
QUALIFY RANK() OVER (PARTITION BY department ORDER BY salary DESC) = 1;
```

Here, `RANK()` ranks employees within each department by salary, and `QUALIFY` filters the results to show only the top earner in each department.

```sql
┌────┬───────┬─────────────┬────────┐
│ id ┆ name  ┆ department  ┆ salary │
╞════╪═══════╪═════════════╪════════╡
│  2 ┆ Bob   ┆ Engineering ┆  72000 │
│  4 ┆ Diana ┆ HR          ┆  68000 │
└────┴───────┴─────────────┴────────┘
```

Previously when doing something like this, I would use a common table expression (CTE):

```sql
WITH ranked_employees AS (
    SELECT 
        *,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS employee_rank
    FROM employees
)

SELECT *
FROM ranked_employees
WHERE employee_rank = 1;
```

As a small query, I still think using a CTE is fine, but when using multiple CTEs in a larger or more complex query, the brevity of `QUALIFY` is really nice.

Want to try it out? Check out the [DuckDB web shell](https://shell.duckdb.org/#queries=v0,CREATE-TABLE-employees-AS%0ASELECT-*-FROM-(VALUES%0A----(1%2C-'Alice'%2C-'Engineering'%2C-70000)%2C%0A----(2%2C-'Bob'%2C-'Engineering'%2C-72000)%2C%0A----(3%2C-'Charlie'%2C-'HR'%2C-65000)%2C%0A----(4%2C-'Diana'%2C-'HR'%2C-68000)%2C%0A----(5%2C-'Eve'%2C-'Engineering'%2C-71000)%0A)-AS-t(id%2C-name%2C-department%2C-salary)~%0A,SELECT-*-FROM-employees~,SELECT-*-FROM-employees-QUALIFY-RANK()-OVER-(PARTITION-BY-department-ORDER-BY-salary-DESC)-%3D-1~).

