# Filtering Dates and Numbers in SQL

> **Disclaimer:** This project was completed as part of the Google Cybersecurity Certificate (Coursera). The scenario and database come from the course. The queries and explanations are my own documentation of the work I performed.

## Project description
I was investigating a recent security incident and needed to gather information about login attempts for specific dates and times. Using SQL in a MariaDB shell, I filtered the `log_in_attempts` table with the `WHERE` keyword, comparison operators, and `BETWEEN ... AND` to narrow down login activity by date, time, and event ID. Filtering data accurately lets an analyst focus on the records that matter in an investigation instead of scanning every row.

## Database Table
**`log_in_attempts`** (user activity)

| Column | Description |
|---|---|
| `event_id` | Unique identifier for the login event (numeric) |
| `username` | Account that attempted to log in |
| `login_date` | Date of the attempt |
| `login_time` | Time of the attempt |
| `country` | Country the attempt came from |
| `ip_address` | IP address of the attempt |
| `success` | Whether the attempt succeeded |

## Operators Used
| Operator | Meaning |
|---|---|
| `=` | Equal to |
| `>` | Greater than |
| `<` | Less than |
| `<>` | Not equal to |
| `>=` | Greater than or equal to |
| `<=` | Less than or equal to |
| `BETWEEN ... AND` | Within a range, including both endpoints |

`WHERE` filters the rows a query returns. Dates and times go in single quotation marks, while numeric data like `event_id` does not.

## Retrieve login attempts after a certain date
To investigate the incident, I first retrieved all login attempts made after May 9, 2022. The greater than operator (`>`) excludes the date itself:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date > '2022-05-09';
```
Result: **125** login attempts.

I then expanded the search to include May 9 by switching to greater than or equal to (`>=`):

```sql
SELECT *
FROM log_in_attempts
WHERE login_date >= '2022-05-09';
```
Result: **165** login attempts.

## Retrieve logins in a date range
Login attempts after May 11, 2022 were not relevant to the incident, so I narrowed the search to a date range with `BETWEEN` and `AND`:

```sql
SELECT *
FROM log_in_attempts
WHERE login_date BETWEEN '2022-05-09' AND '2022-05-11';
```
Result: **123** login attempts.

## Investigate logins at certain times
The organization's typical work hours begin at 07:00:00, so logins before that time are worth a closer look. I first retrieved all attempts made before 07:00:00:

```sql
SELECT *
FROM log_in_attempts
WHERE login_time < '07:00:00';
```

That returned more results than I needed, so I narrowed it to the one-hour window between 06:00:00 and 07:00:00:

```sql
SELECT *
FROM log_in_attempts
WHERE login_time BETWEEN '06:00:00' AND '07:00:00';
```

## Investigate logins by event ID
For this step I only needed three columns: `event_id`, `username`, and `login_date`. I started with every login attempt with an `event_id` of 100 or higher:

```sql
SELECT event_id, username, login_date
FROM log_in_attempts
WHERE event_id >= 100;
```

Then I limited the results to event IDs from 100 to 150:

```sql
SELECT event_id, username, login_date
FROM log_in_attempts
WHERE event_id BETWEEN 100 AND 150;
```

## Findings
| Query | Result |
|---|---|
| Logins after 2022-05-09 (`>`) | 125 attempts |
| Logins on or after 2022-05-09 (`>=`) | 165 attempts |
| Logins between 2022-05-09 and 2022-05-11 | 123 attempts |
| Fifth record with a login time before 07:00:00 | `eraab` |
| Earliest login between 06:00:00 and 07:00:00 | 06:01:31 |
| Third record with `event_id` of 100 or higher | login date 2022-05-09 |
| Seventh record with `event_id` between 100 and 150 | `tmitchel` |

- The difference between `>` and `>=` (165 minus 125) shows that 40 login attempts happened on May 9 itself, which is why the boundary operator matters in an investigation.
- Narrowing the time filter from "before 07:00:00" to the 06:00:00 to 07:00:00 window isolated the logins in the hour just before normal work hours. The earliest in that window was at 06:01:31, a good starting point for checking the account against the `success` column.

## Full Query Sequence
```sql
SELECT * FROM log_in_attempts WHERE login_date > '2022-05-09';
SELECT * FROM log_in_attempts WHERE login_date >= '2022-05-09';
SELECT * FROM log_in_attempts WHERE login_date BETWEEN '2022-05-09' AND '2022-05-11';
SELECT * FROM log_in_attempts WHERE login_time < '07:00:00';
SELECT * FROM log_in_attempts WHERE login_time BETWEEN '06:00:00' AND '07:00:00';
SELECT event_id, username, login_date FROM log_in_attempts WHERE event_id >= 100;
SELECT event_id, username, login_date FROM log_in_attempts WHERE event_id BETWEEN 100 AND 150;
```

## Summary
I used the `WHERE` keyword with comparison operators and `BETWEEN ... AND` to filter login attempts by date, time, and event ID. Each query started broad and was then narrowed to the records relevant to the incident: 125 logins after May 9, 123 logins between May 9 and May 11, and the logins in the hour before normal work hours, the earliest of which was at 06:01:31. Filtering this way is a core skill for incident investigation because it isolates suspicious activity from the rest of the data.

## Key Takeaways
- `WHERE` filters rows, and comparison operators (`>`, `<`, `>=`, `<=`, `=`, `<>`) control exactly which records are included.
- `>` excludes the boundary value, while `>=` includes it. A one-character difference changed the result from 125 to 165 rows.
- `BETWEEN ... AND` is inclusive of both endpoints, which makes it a clean way to filter a date, time, or number range.
- Dates and times go in single quotation marks. Numeric values like `event_id` do not.
- Logins before normal work hours are a common indicator of compromised credentials, so time filters are a useful investigation tool.
- Start with a broad query, then narrow it. If a query returns more than you need, add or tighten a filter.
- Selecting specific columns instead of `*` keeps results focused on the data the investigation needs.

## Tools
- MariaDB shell
- SQL
