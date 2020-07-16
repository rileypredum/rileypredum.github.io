---
layout: post
title:  "Intro to SQL - Part II"
date:   2020-07-15 21:00:00
categories: intros sql
description: A second step into SQL for data analytics

---
A couple weeks ago, I decided to write more on what I’ve learned as a Data Analyst about SQL and analytical thinking. I work for an [edtech startup](https://www.springboard.com/?utm_source=medium&utm_medium=blog&utm_campaign=teach_yourself_sql&utm_term=riley_article) that focuses on getting students jobs in hot fields such as Data Science and UX Design.

This article is the second article in this series (see Part I here). Part I’s goals were to get you set up on Mode Analytics, and to get you familiar with basic SQL on an example database.

In this article, I’ll cover the following:
1. Diving deeper into SQL using sub-queries.
   
2. Tie things back to business example cases and how the results of the queries could be acted upon strategically.
   
3. Show some examples of how to effectively communicate the results of these queries with visualizations.

## Goal 1: recap and more SQL

To recap what was covered in [Part I](https://rileypredum.github.io/intros/sql/intro-to-sql), we got set up in the query editor on Mode, and ran some basic queries against the Yammer experiment tables, working on the following functionalities of SQL: `SELECT`, `FROM`, `LIMIT`, `COUNT`, `GROUP BY`, `ORDER BY`, and `MIN` and `MAX`.

These are great for getting summary statistics and understanding your data. Let’s try to figure out a more complex business problem from the data next! But before I dive into that, I wanted to share a productivity tip:

In most SQL clients (MySQL Workbench, and this GUI Mode analytics platform we’ve been using, among other software) the end of a query is denoted with a semi-colon (;). In the next section, I’m going to write out several queries, with comments (a best practice) to show you that you can have multiple queries in a page and run whichever you want (running CTRL + Enter with it highlighted or with your cursor anywhere between `SELECT` and the semi-colon will run that query only).

Here’s an example of what I mean:

```sql
-- This subquery produces the event type counts
SELECT 
      events.event_type AS event_type,
      COUNT(1) AS number_of_occurrences
FROM 
      tutorial.yammer_events AS events
GROUP BY
      events.event_type
;
-- This subquery produces the count of actions by location
SELECT 
      events.location,
      COUNT(1) AS number_of_occurrences
FROM 
      tutorial.yammer_events AS events
GROUP BY
      events.location
ORDER BY
      COUNT(1) DESC
;
-- This grabs the count of actions by device
SELECT 
      events.device,
      COUNT(1) AS number_of_occurrences
FROM 
      tutorial.yammer_events AS events
GROUP BY
      events.device
ORDER BY
      COUNT(1) DESC
;
```

The two dashes are single-line comments. To comment out multiple lines, begin with ‘/\*’ and end with ‘\*/’. Running the above queries and checking the outputs, we see most of the actions are engagements. The US, Japan, and Germany are the top 3 locations of actions in these data. The most common device is the Macbook Pro, followed by the Lenovo Thinkpad, and finally the Macbook Air.

## Goal 2: Tying it to business example cases

Tying this back to a business case: if there were a bug affecting users on both OS X and Windows, it may be advisable for the tech team to work on the bug fix for OS X first, and roll that out while working on the Windows fix, since these are the two largest numbers of users that could be affected.

A more detailed table might store operating system versions too, which can help the developers know exactly what versions to patch for. Similar logic to the above can be applied when prioritizing which OS version to solve for first if those data were available.

What if we wanted to figure out if users that open emails exhibit other behavior? We need to explore more before we can try to figure out what patterns to look for.

Let’s see what the average number of actions a user takes is. This will give us a sense of the volume of engagement to expect:

```sql
SELECT
      user_id,
      COUNT(1)
FROM 
      tutorial.yammer_events AS events
GROUP BY
      user_id,
      event_name
;
```

Keep in mind that the events table is an actual events table. So all we need to do is count up the user ids and group by user id. Wait, didn’t we want average?
Let’s introduce the concept of a subquery to get that value!

A subquery is a query within a query. It took me a while to understand how to use them until my co-worker explained a heuristic that helped set my mental model right. After that, it just clicked.

When you say `SELECT` something `FROM` table, you’re querying that table and selecting the columns specified in the `SELECT` clause. A sub-query returns what’s called a derived table, which is just a fancy way of saying the output of the query is a table which persists only for the duration of the query. Just like querying any table, you can query from that table, which is itself a query. I’ll start off with an example that wouldn’t make sense in practice but is illustrative of this concept in a simplified way.

```sql
SELECT
      user_id
FROM
(
      SELECT
            user_id
      FROM 
            tutorial.yammer_events AS events
) AS example_derived_table
;
```

This should be clear and illustrate why I learned to tabulate (i.e. indent) in the way I did. The opening parenthesis and closing parenthesis are in line with each other so you know where the sub-query starts and ends easily (trust me with 10 or 20 sub-queries, this is crucial!).

This query makes no sense to do, but it illustrates the point: I’m deriving a table of the user IDs in the events table, and from that, I’m pulling user ID.

Can you guess how we could use this to get the average number of actions a user takes using our previous `COUNT` query?

Get the count of actions by user, and then `FROM` that, take the average of those counts!

```sql
SELECT
      AVG(action_count)
FROM
(
      SELECT
            user_id,
            COUNT(1) AS action_count
      FROM 
            tutorial.yammer_events AS events
      GROUP BY
            user_id
) AS example_derived_table
;
```

This returned 34.9! The first user had only a few though, so I suspect there’s a huge spread of actions, and indeed usually there are highly engaged and barely engaged users. So let’s get the COUNT of counts!

```sql
SELECT
      action_count,
      COUNT(1)
FROM
(
      SELECT
            user_id,
            COUNT(1) AS action_count
      FROM 
            tutorial.yammer_events AS events
      GROUP BY
            user_id
) AS example_derived_table
GROUP BY
      action_count
ORDER BY
      action_count
;
```

This returns to us the count of numbers of actions. The most common is a single action, which makes sense as its easier to take less action than more.

In the above sub-query examples, I sub-queried in the `FROM` clause. But you can sub-query in any clause. I typically will write a sub-query in a `JOIN` clause, because I want to join some manipulated data back onto some base table.

## Goal 3: Effective Visualizations

Oftentimes it is most efficient to produce simple, readable charts to convey information to stakeholders in the business. Say you were working on a product analytics team, and you were tasked with showing the spread of user engagement. How many users perform x number of actions total?

We already have the query that produces this result. You could then make a bar chart out of the resultant data, either in Mode or after extracting the data as a CSV and importing it into Python, Excel, or any other tool you prefer!

In Mode, you simply take the query you’ve written, then click the chart button and choose a bar chart. Drag the columns derived from the query into the X and Y axes, and there’s your chart!

This shows us there are a huge number of users that perform one action, and significantly less that perform more than that. The x-axis is not ideal though, so let’s revisit the query and use the `CASE` statement and predicate logic to create a better bucketing of the data for the X-axis. This will allow us to truncate the data for readability and conciseness.

Here’s the SQL query that buckets the data on the X-axis:

```sql
SELECT
      CASE
        WHEN action_count BETWEEN 0 AND 10 THEN '[0-10]'
        WHEN action_count BETWEEN 10 AND 20 THEN '[10-20]'
        WHEN action_count BETWEEN 20 AND 30 THEN '[20-30]'
        WHEN action_count BETWEEN 30 AND 40 THEN '[30-40]'
        WHEN action_count BETWEEN 40 AND 50 THEN '[40-50]'
        WHEN action_count BETWEEN 50 AND 60 THEN '[50-60]'
        WHEN action_count BETWEEN 60 AND 70 THEN '[60-70]'
        WHEN action_count BETWEEN 70 AND 80 THEN '[70-80]'
        WHEN action_count BETWEEN 80 AND 90 THEN '[80-90]'
        WHEN action_count BETWEEN 90 AND 100 THEN '[90-100]'
      END AS action_count_bucketed,       
      SUM(number_of_users)
FROM
(
    SELECT
          action_count,
          COUNT(1) AS number_of_users
    FROM
    (
          SELECT
                user_id,
                COUNT(1) AS action_count
          FROM 
                tutorial.yammer_events AS events
          GROUP BY
                user_id
    ) AS example_derived_table
    GROUP BY
          action_count
    ORDER BY
          action_count
) AS case_table
GROUP BY
      action_count_bucketed
LIMIT 10
;
```

Much better! Using case statements and capturing the ranges of values, I was able to simplify this chart down. There’s no need to know that 20 or so users performed 98 actions. It’s “good enough” to say that 120 performed 90 to 100 actions! Always keep in mind the 80/20 rule and how much is just enough information.

The story behind this chart is that you can see that most users are not very engaged, with a few core groups in the 10-20 through 30-40 actions buckets. The next step in this is to work with the datetime column occured_at to tell a much more interesting story of when these actions occur, and to look at important product engagement metrics like daily active users (DAU), weekly active users (WAU) and more.

I’ll dive deeper into working with datetime values in the next part of this series, so stay tuned!

If you found this article useful or learned something new, consider donating any amount to pay it forward for the next learner!

Thanks for reading and happy coding!

Riley