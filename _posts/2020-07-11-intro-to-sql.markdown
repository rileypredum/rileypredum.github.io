---
layout: default
title:  "Intro to SQL - Part I"
date:   2020-07-11 10:53:53 -0700
categories: intros sql
description: A first step into SQL for data analytics

---
As a Data Analyst, I work a lot in SQL, and I thought I’d share a walk through on how to use it to solve business problems. I’m self-taught and worked my way from Marketing into a Data Analyst role at an [edtech startup](https://www.springboard.com/?utm_source=github_pages_riley&utm_medium=blog&utm_campaign=teach_yourself_sql&utm_term=riley_article) specializing in Data Science and UX Design education.


I wanted to pay it forward with my learnings and share some of the things I’ve found helpful in developing analytical skills. Without further ado, I’m going to jump in to the meat and potatoes: an end-to-end example of a few business/product analyst style problems you can solve with SQL.

This article is for those of you looking to develop your SQL/analytical thinking skills, and those trying to break into and start their careers as a Business Analyst, Data Analyst, Product Analyst, or Data Scientist. Whatever the path you choose, I guarantee you that SQL is a crucial skill to have in your toolbox!

# Goals
1. Set up your own free instance of Mode, a commonly used business intelligence platform. Many Data Analyst and Data Scientist job descriptions look for candidates who know this tool so it’s good to get familiar with if you’re looking to break into the field.
   
2. Write basic SQL statements against a test data source provided by Mode.
In a future part to this article series, I’ll dive deeper into more advanced SQL, and using it to solve business problems.

# Content 

## Goal 1: Setup
To get started, you’re going to want to go ahead and go to mode.com and sign up for a free account. It’s a super quick and basic [sign-up process](https://app.mode.com/signup). After you’ve verified your account you can log in.

Once in the platform, go to `SQL Training2019` and click `Write a Query`. As someone with experience in product and business analytics I’m going to walk you through SQL queries in the yammer_experiments table, which corresponds to product experiments and their results.

With that, we’ve achieved goal #1! Look at us go!

## Goal 2: Basic SQL
Let’s run the first query to figure out how many rows we’re dealing with:

```sql
SELECT
     COUNT(1)
FROM
     tutorials.yammer_experiments
;
```

The query above tells you the number of rows in the table — 2595.

Now that we know the size of this table, let’s get a sense of the table’s values. To do this, write the following using the `LIMIT` clause:

```sql
SELECT
     *
FROM
     tutorials.yammer_experiments
LIMIT 100
;
```

The asterisk gets all columns and the `LIMIT` clause reduces the number of rows returned, otherwise you’ll be waiting for a while, especially on huge tables.

It looks like the yammer_experiments table is some kind of web analytics data. Seeing the occurred_at field makes me think it’s what’s called an events table. An events table is different in that each row corresponds to an action a user takes. 

Typically there is a trigger set up in the database where the following logic is carried out: if a user takes action A, log a record capturing that user-action pair in events table A. The occurrence of the action activates the trigger and pushes a record to that table.

I don’t know this table though, so this is my hypothesis. In Analyst work, it’s essential to develop and test hypotheses. To test my hypothesis that each row is a user- action pair and therefore any user could perform more than one action, let’s count the number of times each user_id appears and see if there are more than one entry for any user id. This next query introduces two new clauses: `GROUP BY` and `ORDER BY`.

```sql
SELECT
      user_id,
      COUNT(1)
FROM
      tutorial.yammer_experiments
GROUP BY
      user_id
ORDER BY
      COUNT(1) DESC
;
```

In the above query, I selected the user ids from the table and used the same `COUNT(1)` logic introduced above. `GROUP BY` groups the counts by user id. Finally, I used `ORDER BY` and the keyword `DESC` (standing for descending order) to get the largest number of instances of user ids first. This produced the following result:

This invalidates my hypothesis that this is an events table, but let’s soldier onward!

Let’s understand the next column better: `occurred_at`. So we know that a user only shows up once, maybe it’s some kind of action that can’t be performed twice.

At any rate, it would be interesting assuming your stakeholder didn’t tell you this information or you didn’t ask them in a meeting when this experiment was run. It’s easy to do this, using aggregate functions: `MIN` and `MAX`. Take a look and run the query below:

```sql
SELECT
    MIN(occurred_at),
    MAX(occurred_at)
FROM
      tutorial.yammer_experiments
;
```

This will show us that the earliest action was performed June 1st, 2014 just past midnight and the latest action was 10:45 PM on June 30th, 2014. Let’s assume this was a June 2014 experiment then!

A note on readability and ease of understanding is needed now. Using the AS keyword in our `SELECT` clause for these columns is a little nicer for our own understanding and anyone who might run this query in the future.

```sql
SELECT
    MIN(occurred_at) AS `Earliest Occurrence`,
    MAX(occurred_at) AS `Latest Occurrence`
FROM
      tutorial.yammer_experiments
;
```

This is called aliasing, and it helps with readability and understanding in SQL. When working with other analysts solely however, I’d actually recommend keeping the table and field names exactly as is — because aliases are your own naming conventions. 

Other analysts will only readily understand the database schema as it’s actually called and not what you decide to rename it! So if you’re looking for feedback or working collaboratively, try not to alias even though you’ll be typing more. Be a nice fellow Analyst — don’t alias.

Note: backticks (`) don’t work in every version of SQL for aliasing, and some SQL variants have a hard time with spaces. Experiment with underscores instead of spaces, single, or double quotes until you don’t get an error if that’s the case.

Before diving into other ways to explore these data, it’s important to see if the control and test groups are equivalent sizes. If we are doing an A/B test looking at two variants of a design flow, button clicking behavior, landing page, etc., we want the data to be evenly split across the number of variants. Let’s check that with the `COUNT(1)` trick again.

```sql
SELECT
    experiment_group,
    COUNT(1) AS number_of_participants
FROM
      tutorial.yammer_experiments
GROUP BY
    experiment_group
;
```

This shows us that while test_group had 849 participants, control_group had 1746 participants! While this split is not ideal, both sides do have statistically significant numbers of observations, so let’s take note of that A/B split caveat and push onward — this is the kind of thing to note to stakeholders and add as a recommendation for the future — split your groups evenly, please!

You may have noticed in Mode they have multiple Yammer tables. Were you wondering how they relate? A Data org worth its salt would have an ER (entity relationship) Diagram, which shows the primary and foreign keys of tables. 

Primary keys are usually the first column in a table and foreign keys are the keys that point outward from that table, mapping to the primary keys of whatever other table you’re trying to `JOIN` to. This brings me to the concept of the `JOIN`.


Let’s try the following: `JOIN` the users table to the experiments table. Before we dive in, and since we don’t know the schema, let’s learn the schema empirically by running queries against the tables! To aid in readability and deal with weird linebreaks from table names being too long, I’ll break my own rule and alias the next section’s tables and fields.

Let’s see if all the users were used in the experiment table:

```sql
SELECT
      COUNT(1)
FROM
      tutorial.yammer_users AS users
INNER JOIN
      tutorial.yammer_experiments AS experiments 
      ON experiments.user_id = users.user_id
;
```

The query above again counts the number of rows, but the number of rows that are returned after all user ids in the users table are inner joined to the experiments table. The result? 2595. Why? `INNER JOIN` is the inner section of a venn-diagram, meaning that records in table A will only be returned if they match up to the field used in the ON clause in table B (experiments table). 

In plain English: 2595 rows are returned because out of all the users (the users table), only these 2595 user IDs were used in the experiments table.

Because the experiments table is a subset of the users table (rightly so as you don’t want to run an experiment against all your users!), this is a great opportunity to use another kind of join: `LEFT JOIN`.

Let’s try the same query as above but using a `LEFT JOIN` instead of an `INNER JOIN`:

```sql
SELECT
      COUNT(1)
FROM
      tutorial.yammer_users AS users
LEFT JOIN
      tutorial.yammer_experiments AS experiments 
      ON experiments.user_id = users.user_id
;
```

Well? Did it work? It should have returned the number of rows in the users table! This is because it takes all user IDs in the users table, and only the rows in the experiments table for which the user IDs correspond. Dis-aggregate the table produced by the above query to see for yourself!

```sql
SELECT
      *
FROM
      tutorial.yammer_users AS users
LEFT JOIN
  tutorial.yammer_experiments AS experiments 
  ON experiments.user_id = users.user_id
;
```

![test image](https://rileypredum.github.io/assets/images/image.png "test")


What happened? I returned all columns in both tables, and where a user ID was in both the users table and the 
experiments table, the columns in the experiments table are populated with that user’s corresponding experiments data, otherwise the values are NULL because there was no match!

## Conclusion
In this first part of this SQL Academy series, you learned the basics of SQL: `SELECT`, `FROM`, `LIMIT`, `COUNT`, `GROUP BY`, `MIN`, `MAX`, and `ORDER BY`.

In the next part I wrote about sub-queries and case statements. Check it out here!

If you found this article useful or learned something new, consider donating any amount to pay it forward for the next learner!

Thanks for reading and happy coding!

Riley