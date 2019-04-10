
# Grouping Data with SQL

## Introduction

Another very useful tool in SQL is the ability to run aggregate functions. For example, in the customer database below, you might want to look at mean or median sales to compare them across offices or regions.

## Objectives

You will be able to:

* Write queries with aggregate functions like `COUNT`, `MAX`, `MIN`, and `SUM`
* Create an alias for the return value of an aggregate function
* Use `GROUP BY` to sort the data sets returned by aggregate functions
* Compare aggregates using the `HAVING` clause

## Database Schema
<img src="images/Database-Schema.png">


```python
import sqlite3
import pandas as pd
```

## Connecting to the Database

As usual, start by creating a connection to the database.


```python
conn = sqlite3.Connection('data.sqlite')
cur = conn.cursor()
```

## Groupby and Aggregate Functions

Lets start by looking at some `GROUP BY` statements to aggregate our data.


```python
#Here we join the offices and employees tables in order to count the number of employees per city.
cur.execute("""SELECT city,
                      COUNT(employeeNumber)
                      FROM offices
                      JOIN employees
                      USING(officeCode)
                      GROUP BY city
                      ORDER BY COUNT(employeeNumber) DESC;""")
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df.head()
```

## Aliasing
You can also alias our groupby by specifying the number of our selection order that we want to group by. This is simply written as `group by 1` 1 referring to the first column name that we are selecting.

Additionally, we can also rename our aggregate to a more descriptive name using the `as` clause.


```python
cur.execute("""SELECT city,
                      COUNT(employeeNumber) as numEmployees
                      FROM offices
                      JOIN employees
                      USING(officeCode)
                      GROUP BY 1
                      ORDER BY numEmployees DESC;""")
df = pd.DataFrame(cur.fetchall())
df.columns = [x[0] for x in cur.description]
df.head()
```

## Other Aggregations

Aside from count() some other useful aggregations include:
    * min()
    * max()
    * sum()
    * avg()


```python
cur.execute("""SELECT customerName,
                      COUNT(*) as number_purchases,
                      MIN(amount) as min_purchase,
                      MAX(amount) as max_purchase,
                      AVG(amount) as avg_purchase,
                      SUM(amount) as total_spent
                      FROM customers
                      JOIN payments
                      USING(customerNumber)
                      GROUP BY 1
                      ORDER BY SUM(amount) DESC;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```


```python
df.tail()
```

## The having clause

Finally, we can also filter our aggregated views with the having clause. The having clause works like the where clause but is used to filter data selections on conditions post the group by. For example, if we wanted to filter based on a customer's last name, we would use the where clause. However, if we wanted to filter a list of city's with at least 5 customers, we would using the having clause; we would first groupby city and count the number of customers, and the having clause allows us to pass conditions on the result of this aggregation.


```python
cur.execute("""SELECT city,
                      COUNT(customerNumber) AS number_customers
                      FROM customers
                      GROUP BY 1
                      HAVING COUNT(customerNumber)>=5;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```

## Combining the where and having clause
We can also use the where and having clause in conjunction with each other for more complex rules.
For example, let's say we want a list of customers who have made at least 3 purchases of over 50K each.


```python
cur.execute("""SELECT customerName,
                      COUNT(amount) AS number_purchases_over_50K
                      FROM customers
                      JOIN payments
                      USING(customerNumber)
                      WHERE amount >= 50000
                      GROUP BY 1
                      HAVING COUNT(amount) >= 3
                      ORDER BY COUNT(amount) DESC;""")
df = pd.DataFrame(cur.fetchall())
df. columns = [i[0] for i in cur.description]
print(len(df))
df.head()
```


```python
df.tail()
```

## Summary

After this section, you should have a good idea of how to use aggregate functions, aliases and the `HAVING` clause to filter selections.
