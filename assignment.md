# Introduction to SQL

## Introduction
#### Goals

   1. Explore the Database
   2. Basic Querying - Selecting From Tables
   3. Selecting specific attributes of a table
   4. Where clause/ filtering
   5. Aggregation functions: counting
   6. Aggregation functions: AVG
   7. Intervals, Ranges, and sorting
   8. Subqueries


## Basic
### Part 1: Loading the database

In this repo, there's a SQL dump of the data we'll be using today. This data is typical of the type of data you might encounter in industry. It is [normalized](https://en.wikipedia.org/wiki/Database_normalization), which is a way of minimizing the disk space required to store the data and ensuring consistency, but which can sometimes require more effort to get data, since most queries will require information stored across multiple tables. As an example, the events table has ids for both users and meals, but in order to get the price of the meal, we have to look up that meal in the id table. This allows us to use _only_ the id to refer to the meal _anywhere_ it may appear, but does mean that to get meal details we almost always have to join. *As a data scientist, you will be writing a lot of SQL in order to get data from various tables into a single location where you can use it.*

1. If you are on your personal computer and haven't set up postgres yet, follow [these instructions](https://github.com/GalvanizeDataScience/docker/blob/master/reference/docker_postgres.md) to start a docker container.

1. **From a Docker bash shell** run `psql -U postgres` and then this command to create the database. See [here for more details on loading the data](https://github.com/GalvanizeDataScience/docker/blob/master/guides/docker_postgres.md)

    ```sql
    CREATE DATABASE readychef;
    \q
    ```

1. **Back in the Docker bash shell** navigate to where the data on your mounted folder is:

    ```
    cd data
    psql -U postgres readychef < readychef.sql
    ```

    You should see a bunch of SQL commands flow through the terminal. 

1. To enter the interactive Postgres command-line client and connect to your database, enter the following from the bash shell in the docker container.

    ```
    psql -U postgres readychef
    ```
This will put you in the command-line client. We won't use that for this assignment, but you can type

    ```sql
    SELECT * FROM users LIMIT 10; 
    ```

to see the first rows of the users table and verify the database was set up correctly (don't forget the semicolon!). We can do
    ```
    \d
    ```
in the command-line client to list all the tables in the database, and look at details of the user table with
    ```
    \d users
    ```

When you've looked at the details for each table, you can enter
    ```
    \q
    ```

to exit the command-line client, but you might keep this open to refer to it again.

### Part 2: Basic Exploration

For this assignment we'll access the postgres database from python using a jupyter notebook.

1. Import `psycopg2` and create a connection to the database with

    ```python
    conn = psycopg2.connect(dbname='readychef',
                        host='localhost',
                        user='postgres',
                        password='password')
    ```
2. Create a cursor from the connection and execute a query to select the first 10 rows from the event table. Use a `for` loop to iterate over the cursor, printing out each row.

3. Write a function that takes a query string and database connection, executes the query, and returns a list of rows. Call the function on a statement selecting the first 10 rows of the users table. Is it better for a function to return a list of rows or the cursor itself?

4. Use the `read_sql` function in `pandas` to load the same query into a dataframe. For the remainng questions, you can use either approach. 

### Part 3: Select statements

1. To get an understanding of the data, run a [SELECT](http://www.postgresqltutorial.com/postgresql-select/) statement on each table. Keep all the columns and limit the number of rows to 10.

2. Write a `SELECT` statement that would get just the userids from the events table.

3. Maybe you're just interested in what the campaign ids are. Use 'SELECT DISTINCT' to figure out all the possible values of that column in the appropriate table.

    *Note:*  Pinterest=PI, Facebook=FB, Twitter=TW, and Reddit=RE

### Part 4: Where Clauses / Filtering

Now that we have the lay of the land, we're interested in the subset of users that came from Facebook (FB). If you're unfamiliar with SQL syntax, the [WHERE](http://www.postgresqltutorial.com/postgresql-where/) clause can be used to add a conditional to `SELECT` statements. This has the effect of only returning rows where the conditional evaluates to `TRUE`. 

*Note: Make sure you put string literals in single quotes, like `campaign_id='TW'`.*

1. Using the `WHERE` clause, write a new `SELECT` statement that returns all rows where `Campaign_ID` is equal to `FB`.

2. We don't need the campaign id in the result since they are all the same, so only include the other two columns.

    Your output should be something like this:

    ```
userid	dt
0	3	2013-01-01
1	4	2013-01-01
2	5	2013-01-01
3	6	2013-01-01
4	8	2013-01-01
...
    ```

### Part 5: Aggregation Functions

Let's try some [aggregation functions](http://www.postgresql.org/docs/8.2/static/functions-aggregate.html) now.

`COUNT` is an example aggregate function, which counts all the entries and you can use like this:

```sql
SELECT COUNT(*) FROM users;
```

Your output should look something like:

```
    count
0	5524

```

1. Write a query to get the count of just the users who came from Facebook.

2. Now, count the number of users coming from each service. Here you'll have to group by the column you're selecting with a [GROUP BY](http://www.postgresql.org/docs/8.0/static/sql-select.html#SQL-GROUPBY) clause.

    Try running the query without a group by. Postgres will tell you what to put in your group by clause!

3. Use `COUNT (DISTINCT columnname)` to get the number of unique dates that appear in the `users` table.

4. There's also `MAX` and `MIN` functions, which do what you might expect. Write a query to get the first and last registration date from the `users` table.

5. Calculate the mean price for a meal (from the `meals` table). You can use the `AVG` function. Your result should look like this:

    ```
	avg
0	10.652283
    ```

6. Now get the average price, the min price and the max price for each meal type. Don't forget the group by statement!

    Your output should look like this:

    ```
	type	avg	min	max
0	mexican	9.697595	6	13
1	italian	11.292614	7	16
2	chinese	9.518717	6	13
3	french	11.542000	7	16
4	japanese	9.380488	6	13
5	vietnamese	9.283019	6	13
    ```

7. It's often helpful for us to give our own names to columns. We can always rename columns that we select by doing `AVG(price) AS avg_price`. This is called [aliasing](http://stackoverflow.com/questions/15413735/postgresql-help-me-figure-out-how-to-use-table-aliases). Alias all the above columns so that your table looks like this:

    ```
type	avg_price	min_price	max_price
0	mexican	9.697595	6	13
    ...
    ```

8. Maybe you only want to consider the meals which occur in the first quarter (January through March). Use `date_part` to get the month like this: `date_part('month', dt)`. Add a `WHERE` clause to the above query to consider only meals in the first quarter of 2013 (month<=3 and year=2013).

9. There are also scenarios where you'd want to group by two columns. Modify the above query so that we get the aggregate values for each month and type. You'll need to add the month to both the select statement and the group by statement.

    It'll be helpful to *alias* the month column and give it a name like `month` so you don't have to call the `date_time` function again in the `GROUP BY` clause.

    Your result should look like this:
```
       type	month	avg_price	min_price	max_price
0	chinese	1.0	11.230769	8	13
1	chinese	2.0	9.066667	6	13
2	chinese	3.0	9.250000	6	13
3	french	1.0	11.650000	7	16
```

10. From the `events` table, write a query that gets the total number of buys, likes and shares for each meal id. 
_Extra_: To avoid having to do this as three separate queries you can do the count of the number of buys like this: `SUM(CASE WHEN event='bought' THEN 1 ELSE 0 END)`.

## Advanced

### Part 6: Sorting

1. Let's start with a query which gets the average price for each type. It will be helpful to alias the average price column as 'avg_price'.

2. To make it easier to read, sort the results by the `type` column. You can do this with an [ORDER BY](http://www.postgresqltutorial.com/postgresql-order-by/) clause.

3. Now return the same table again, except this time order by the price in descending order (add the `DESC` keyword).

3. Sometimes we want to sort by two columns. Write a query to get all the meals, but sort by the type and then by the price. You should have an order by clause that looks something like this: `ORDER BY col1, col2`.

4. For shorthand, people sometimes use numbers to refer to the columns in their order by or group by clauses. The numbers refer to the order they are in the select statement. For instance `SELECT type, dt FROM meals ORDER BY 1;` would order the results by the `type` column.

### Part 7: Joins

Now we are ready to do operations on multiple tables. A [JOIN](http://www.tutorialspoint.com/postgresql/postgresql_using_joins.htm) allows us to combine multiple tables.

1. Write a query to get one table that joins the `events` table with the `users` table (on `userid`) to create the following result.

    ```
	userid	campaign_id	meal_id	event
0	3	FB	18	bought
1	7	PI	1	like
2	10	TW	29	bought
3	11	RE	19	share
    ...
    ```

2. Also include information about the meal, like the `type` and the `price`. Only include the `bought` events. The result should look like this:

    ```
  	userid	campaign_id	meal_id	type	price
0	3	FB	18	french	9
1	10	TW	29	italian	15
2	18	TW	40	japanese	13
3	22	RE	23	mexican	12
4	25	FB	8	french	
    ...
    ```

    If your results are different, make sure you filtered it so you only got the `bought` events. You should be able to do this *without* using a where clause, only on clause(s)! (note this has no effect on performance, and may make the query more confusing)

3. Write a query to get how many of each type of meal were bought.

    You should again be able to do this *without* a where clause!

*Phew!* If you've made it this far, congratulations! You're ready to move on to subqueries.

### Part 8: Subqueries

In a [subquery](http://www.postgresql.org/docs/8.1/static/functions-subquery.html), you have a select statement embedded in another select statement.

1. Write a query to get meals that are above the average meal price.

    Start by writing a query to get the average meal price. Then write a query where you put `price > (SELECT ...)` (that select statement should return the average price).

2. Write a query to get the meals that are above the average meal price *for that type*.

    Here you'll need to use a join. First write a query that gets the average meal price for each type. Then join with that table to get ones that are larger than the average price for that meal. Your query should look something like this:

    ```sql
    SELECT meals.*
    FROM meals
    JOIN (SELECT ...) average
    ON ...
    ```

    Note that you need to fill in the select statement that will get the average meal price for each type. We *alias* this table and give it the name `average` (you can include the `AS` keyword, but it doesn't matter).

3. Modify the above query to give a count of the number of meals per type that are above the average price.

4. Calculate the percentage of users which come from each service. This query will look similar to #2 from aggregation functions, except you have to divide by the total number of users.

    Like with many programming languages, dividing an int by an int yields an int, and you will get 0 instead of something like 0.54. You can deal with this by casting one of the values as a real like this: `CAST (value AS REAL)`

    You should get a result like this:

    ```
     campaign_id |      percent
    -------------+-------------------
     RE          | 0.156046343229544
     FB          | 0.396813902968863
     TW          | 0.340695148443157
     PI          | 0.106444605358436
    (4 rows)
    ```

## Extra Credit

### Part 9: More practice

1. Answer the question, _"What user from each campaign bought the most items?"_

    It will be helpful to create a temporary table that contains the counts of the number of items each user bought. You can create a table like this: `CREATE TABLE mytable AS SELECT...`

2. For each day, get the total number of users who have registered as of that day. You should get a table that has a `dt` and a `cnt` column. This is a cumulative sum.

3. What day of the week gets meals with the most buys?

4. Which month had the highest percent of users who visited the site purchase a meal?

5. Find all the meals that are above the average price of the previous 7 days.

6. What percent of users have shared more meals than they have liked?

7. For every day, count the number of users who have visited the site and done no action.

8. Find all the dates with a greater than average number of meals.

9. Find all the users who bought a meal before liking or sharing a meal.
