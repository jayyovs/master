# sql Style Guide


## *Introduction*

Writing sql code to materialize dbt data models is an ongoing, collaborative process. Adhering to a style guide leads to:

- sql code that is easy to read and maintain
- A more efficient code review and QA process
- Clear patterns in the codebase, promoting adherence to DRY (Don’t Repeat Yourself) principles

## *General Principles*

- It is our shared responsibility to adhere to and uphold this style guide. The expectation is that the team will follow the guidelines during code development, code reviews, and, most importantly, for any code committed to the GitHub repository
- Our primary goal is to prioritize readability, maintainability, and robustness, rather than minimizing the number of lines of code. Extra lines of code are inexpensive; time is what we need to optimize

## *High-Level Style Guide Rules

Here's example query to demonstrate how this style gfuid is applied in practive:

```sql
with subscriptions as (
  select
    -- first column should always be the primary key
    subs.sf_id as subscription_id, 
    -- the timestamps are suffixed with `_at`
    subs.created_date::timestamp as subscription_created_at, 
    -- the dates are suffixed with `_date`
    subs.start_date::date as subscription_start_date, 
    -- for case statements, separate the when statements for readability
    case
      when subs.plan = 'Basic' then 1
      when subs.plan = 'Premium' then 2
      else 0
    end as plan_enum
  -- Use recognizable aliases. This helps with debugging/reviewing the code
  from stgsf_subscriptions as subs
  left join mart_users as user
    -- the column to be joined will come on the right side
    on subs.user_id = user.id 
  where 1 = 1 -- helps when other clauses need to be commented out for testing
    -- Multiple where clauses should be on separate lines
    and subs.start_date::date >= '2024-01-01' 
    and user.subscription_status is not null
),

subscriptions_sample as ( -- Use descriptive CTE names
  select 
    1 as subscription_id, 
    created_at as subscription_created_at,
    created_at::date as subscription_start_date
  from subscription_csv
),

combined_subscriptions as (
  select * from subscriptions
  union all -- Make use of Union All instead of Union to avoid filtering out rows
  select * from subscriptions_sample
),

final as ( -- lable your last CTE 'Final'
  select
    a.subscription_id,
    coalesce(b.subscription_start_date, a.subscription_start_date) as subscription_start_date,
    -- Boolean fields should be prefixed with `was_`, `is_`, `has_`, or `does_`. For example, `is_active`, `has_unsubscribed`, etc.
    iff(b.status = 'Cancelled', 1, 0))::boolean as is_cancelled, 
  from combined_subscriptions as a
  full outer join stgsf_subscriptions_old as b
    on a.subscription_id = b.sf_id
    and a.subscription_start_date = b.created_date
  group by 1, 2
  -- Use qualify to dedupe rows instead of creating extra CTE
  qualify row_number() over (partition by a.subscription_id order by subscription_start_date desc)
)

select * from final
```

## *Code Formatting*

### Use lowercase sql
It's just as readable as uppercase sql and you won't have to constantly be holding down a shift key.

### Single line vs multiple line queries

The only time to put all of your sql on one line is when you're selecting:

* All columns (*) or selecting a few columns (~ <5)
* _And_ there's no additional complexity in your query

The reason for this is simply that it's still easy to read this when everything is on one line. But once you start adding more columns or more complexity, it's easier to read if it's on multiple lines.

Use trailing commas (as opposed to leading commas)
Commas should be followed by a space before the column name and the first column name should be aligned for readability:

```sql
-- Bad (too many columns on one line)
select id, email, created_at, is_customer, is_unsubscribed
from mart_users

-- Bad (id should be on a new line as well)
select id,
    email
from mart_users

-- Good
select
  id,
  email,
  created_at
from mart_users

-- Good
select *
from mart_users
where email = 'example@domain.com'
```

### Left-align all command words

- When using `select`, indent with a single tab
	- Align `from`, `join`, `where`, and `group by` clauses to match the `select` indentation (i.e. one tab from the left edge)
- For queries with multiple conditions in a `join` or `where` clause, place each condition on a separate line and indent each by one tab

```sql
-- Bad
select
id,
email
  from mart_users
 where email like '%@gmail.com'

-- Good
select
  id,
  email
from users
where mart_users like '%@gmail.com'

-- Bad
select
  id,
  email
from mart_users
where 1=1
and email like '%@gmail.com'
and state = 'CA'

-- Good
select
  id,
  email
from mart_users
where 1=1
  and email like '%@gmail.com'
  and state = 'CA'
```

### Use single quotes

While some sql dialects like BigQuery allow double quotes, most dialects treat double quotes as referring to column names. To avoid ambiguity, it's preferable to use single quotes.

```sql
-- Bad
select *
from mart_users
where email = "example@domain.com"

-- Good
select *
from mart_users
where email = 'example@domain.com'
```

### Use != over <>

Use `!=` instead of `<>` because `!=` reads "not equal," which is is more natural and closer to how we'd say it out loud.

```sql
-- Bad
select *
from stgsf_subscriptions
where plan <> 'Basic'

-- Good
select *
from stgsf_subscriptions
where plan != 'Basic'
```

### Indenting where conditions

When there is only one where condition, leave it on the same line as `where`.

```sql
select *
from stgsf_subscriptions
where plan = 'Premium'
```

When there are multiple conditions, indent each one an additional level beyond the WHERE clause. Use 'where 1 = 1' to make testing conditions easier. Place logical operators (e.g., AND, OR) at the beginning of each condition.

```sql
select *
from stgsf_subscriptions
where 1 = 1
    and plan = 'Premium'
    and start_date::date >= '2024-01-01'
```

### Avoid spaces inside of parenthesis

```sql
-- Bad
select *
from mart_users
where user_id in ( 1, 2 )

-- Good
select *
from mart_users
where user_id in (1, 2)
```

### Use spaces after commas and before and after operators

```sql
-- Bad
select
  id,
  first_name||' '||last_name as full_name,
  coalesce(lifetime_age_yrs,0) as lifetime_age_yrs,
  opening_base_arr-closing_base_arr as net_arr
from mart_users

-- Good
select
  id,
  first_name || ' ' || last_name as full_name,
  coalesce(lifetime_age_yrs, 0) as lifetime_age_yrs,
  opening_base_arr - closing_base_arr as net_arr
from mart_users
```

### Column order conventions

Place the primary key first, followed by foreign keys, then timestamps (e.g., `created_at`, `updated_at`,`bound_at`,`cancellation_at`). If the table inlcudes any boolean flags (e.g., `is_active`, `is_cancelled`) position those at the end.

```sql
-- Bad
select
  subscription_id,
  id,
  subscription_created_at,
  subscription_start_date,
  is_active,
  is_cancelled
from mart_users
where mart_subscriptions

-- Good
select
  subscription_created_at,
  subscription_start_date,
  is_active,
  is_cancelled,
  id,
  subscription_id
from mart_users
where mart_subscriptions
```

### Column names should be snake_case

Column names should use the snake_case convention, where all letters are lowercase and words are separated by underscores `_`.

```sql
-- Bad
select
  id as SubscriptionId,
  date_trunc('mon', start_date) as SubscriptionStartMonth
from stgsf_subscriptions

-- Good
select
  id as subscription_id,
  date_trunc('mon', start_date) as subscription_start_month
from stgsf_subscriptions
```

### Column name conventions

* Boolean fields should be prefixed with `was_`, `is_`, `has_`, or `does_`. For example: `is_customer`, `has_unsubscribed`.
* Aggregation fields should be prefixed with `count_`, `sum_`, or `max_`. For example: `count_subscribers`, `max_liftime_age`.
* Date-only fields should be suffixed with `_date`. For example: `cancellation_date`.
* Date-part fields should be suffixed with `_{date_part}`. For example, `cancellation_month`.
* Date+time fields should be suffixed with `_at`. For example, `created_at`, `cancellation_at`.
* Primary or Surrogate columns should be suffixed with `_key` when being created as a surrogate key.

### Always alias table names when using multiple tables

When working with multiple tables in a query, always alias the table names to improve readability and avoid ambiguity. If you decide to alias with single letters, make sure to start with `a`. If only one table is being used, it is acceptable to omit the alias.

```sql
-- Bad
select
  id,
  name
from mart_subscriptions as a
left join mart_users as b
on a.user_id = b.id

-- Bad
select
  z.id,
  l.name
from mart_subscriptions as z
left join mart_users as l
on a.user_id = b.id

-- Good
select
  a.id,
  b.name
from mart_subscriptions as a
left join mart_users as b
on a.user_id = b.id

-- Good
select *
from mart_subscriptions
```

### Always rename aggregates and function-wrapped arguments

```sql
-- Bad
select
  sum(a.subscription_amount)
from mart_subscriptions as a
left join mart_users as b
on a.user_id = b.id 
where b.state = 'MA'

-- Good
select
  sum(a.subscription_amount) as sum_subscription_amount
from mart_subscriptions as a
left join mart_users as b
on a.user_id = b.id 
where b.state = 'MA'

-- Bad
select
  date_trunc('year', subscription_created_at)::date
from mart_subscriptions

-- Good
select
  date_trunc('year', subscription_created_at)::date as subscription_created_year
from mart_subscriptions
```

### Use `as` to alias column _and_ table names

```sql
-- Bad
select
  iff(a.plan = 'Premium') is_premium
from mart_subscriptions a

-- Good
select
  iff(a.plan = 'Premium') as is_premium
from mart_subscriptions as a
```

### Take advantage of lateral column aliasing

Instead of recycling logic from the `select` clause in a later step (e.g., `group by`), reference the alias for simplicity and scalability.

```sql
-- Bad
select
  date_trunc('year', subscription_start_date) as subscription_start_year,
  sum(arr) as total_arr
from mart_subscriptions
group by date_trunc('year', subscription_start_date)

-- Good
select
  date_trunc('year', subscription_start_date) as subscription_start_year,
  sum(arr) as total_arr
from mart_subscriptions
group by subscription_start_year
```

### Use meaningful CTE names

CTE names should be **descriptive** of what they represent

```sql
-- Bad
with cte as (
  select *
  from mart_users
)

-- Good
with users as (
  select *
  from mart_users
)
```

## *Code Conventions*

### Be explicit in boolean conditions

```sql
-- Bad
select *
from mart_users
where is_active

-- Bad
select *
from mart_users
where not is_active

-- Good
select *
from mart_users
where is_active = true

-- Good
select *
from mart_users
where is_active = false
```

### Group by either column name _or_ ordinal position.

```sql
-- Bad
select
  sales_rep,
  plan,
  avg(discount_amount) as avg_discount
from mart_discounts
group by 1, plan

-- Good
select
  sales_rep,
  plan,
  avg(discount_amount) as avg_discount
from mart_discounts
group by sales_rep, plan

-- Good
select
  sales_rep,
  plan,
  avg(discount_amount) as avg_discount
from mart_discounts
group by 1, 2
```

### Grouping columns should go first

```sql
-- Bad
select
  count(user_id) as total_users,
  plan
from mart_subscriptions
group by plan

-- Good
select
  plan
  count(user_id) as total_users
from mart_subscriptions
group by plan
```
### Aligning case/when statements

Each `when` clause should be on its own line with nothing on the `case` line. Each `when` should be indented on level deeper than the `case` line. The `then` clause can either be on the same line as the `when` or on its own line below it, but aim for consistency through the query.

```sql
-- Bad
select
  case when stage_name = 'accepted lead' then 'Sales Accepted'
      when stage_name = 'qualified opp' then 'Sales Qualified'
      else stage_name
  end as opportunity_stage
from stgsf_opportnity

-- Better
select
  case
    when stage_name = 'accepted lead' then 'Sales Accepted'
      then 'Sales Accepted'
    when stage_name = 'qualified opp' then 'Sales Qualified'
      then 'Sales Qualified'
    else stage_name
  end as opportunity_stage
from stgsf_opportnity

-- Best
select
  case
    when stage_name = 'accepted lead' then 'Sales Accepted'
    when stage_name = 'qualified opp' then 'Sales Qualified'
    else stage_name
  end as opportunity_stage
from stgsf_opportnity

```

### Use CTEs, not subqueries

- Avoid using subqueries; Common Table Expressions (CTEs) make your queries easier to read and understand.
- When using CTEs, pad the query with new lines.
- Always include a CTE named `final` and use `select * from final` at the end of your query. This allows you to quickly inspect the output of others CTEs for debugging purposes.
- Ensure that the closing parenthese for CTEs are indented at the same level ad the `with` keyword and the CTE names.

```sql
-- Bad
select user_id
from (
  select
    user_id,
    row_number() over (partition by user_id order by created_date desc) as transaction_rank
  from int_transactions
) ranked
where last_transaction_date = 1

-- Good
with transaction_details as (
  select
    user_id,
    row_number() over (partition by user_id order by created_date desc) as transaction_rank
  from int_transactions
  from int_transactions
),

final as (
  select
    user_id,
    name
  from transaction_details
  where transaction_rank = 1
)

select * from final
```

### Window functions

- You can either keep the window function on its own line or break it up into muliple lines depending on its length and complexity.
- Use the `qualify()` function to deduplicate or rank rows within partitions.

```sql
-- Bad
with customer_case_rank as (
select
  bsd.user_id,
  bsd.name,
  row_number()
    over (
      partition by customer order by created_date desc
    ) as customer_case_rank
from int_cases
)
select * 
from customer_case_rank
where customer_case_rank = 1

-- Better
with customer_case_rank as (
select
  bsd.user_id,
  bsd.name,
  row_number() over (partition by customer order by created_date desc) as customer_case_rank
from int_cases
)
select * 
from customer_case_rank
where customer_case_rank = 1

-- Best
select
  case_id,
  customer    
from int_cases
qualify row_number() over (partition by customer order by created_date desc) = 1
```

### Additional guidelines

- Prefer `union all` to `union` and `full outer join`
	- `union all` should be used over `union` because `union` removes duplicate records, which can indicate an upstream data integrity issue. If duplicated are to be removed, it is usally best to address earlier in the data model.
  - `full outer join` is often more expensive and can be avoided if possible. In many cases alternative join types or restructuring the query can achieve the desired result with better performance.

- Avoid using `order by` in data models unless necessary
	- The `order by` clause can significantyl affect query performance, so it should only be used if the ordering of the results is essential for generating correct results.
  - Consumers of the query should be able to handle any sorting requirements themselves, so sorting should not be a part of the data modeling process unless explicitly required for accuracy.
 
## *JOIN Statement Conventions*

### Prefix your column names with the table alias

If only selecting from one table, prefixes are not needed

### Never use `using` in joins because it produces inaccurate results in Snowflake

### Always be explicit with join types

Specifying `inner` makes it easier to identify this join type for code review and bug fixing

### Every join condition should be on a new indented line after the join

```sql
-- Bad
select
  a.email,
  sum(b.discount_amount) as total_discounts
from mart_users as a
join mart_discounts as b on a.id = b.user_id
group by 1

-- Good
select
  a.email,
  sum(b.discount_amount) as total_discounts
from mart_users as a
inner join mart_discounts as b
  on a.id = b.user_id
group by 1
```

### Referenced table should come first in join condition

For clarity and easier debugging, always put the referenced table (the table that you are joining to) first immediately after the `on` clause. This makes it easier to understand if the join could potentially cause the results to fan out and helps with identifying the direction of the join.

```sql
-- Bad
select a.*
from stgsf_subscriptions as a
left join mart_users as b
  on b.id = a.user_id

-- Good
select a.*
from stgsf_subscriptions as a
left join mart_users as b
  on a.user_id = b.id
```

## *Commenting*
- For newer or more complicated sql statements, it's reccomended to use comments to explain any complex or less commonly used sql logic. This provides helpful context for yourself in the future or for other team members to better understand the query.
- When commenting multiple lines within a sql model, always use the `/* */` syntax
- Respect the character line limit when making comments
	- If a comment is too long to fit comfortable on one line, break it up and continue on the next line to maintain readability and avoid horizontal scrolling
