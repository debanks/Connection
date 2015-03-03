# MySQL Connection Class

An easy class for handling prepared statements in PHP using mysqli. The class is fully commented
so here I won't explicitly tell what each function is for, just examples of how to use it in 
your own code.

## Quick Links

1. [Insert Statements] (https://github.com/debanks/Connection#1-insert-statements)
2. [Update Statements] (https://github.com/debanks/Connection#2-update-statements)
3. [Select Statements] (https://github.com/debanks/Connection#3-select-statements)
4. [Delete Statements] (https://github.com/debanks/Connection#4-delete-statements)

## What this Class Does

This class is meant to provide interaction to a DB that is already set up how you want it. I have
not found a real need to change the table structure on the fly, so this class provides no functions
for altering, creating, or deleting tables.

So this class handles prepared select, update, insert, and delete functions and covers most cases
for these statements.

## Examples

```php
$db = new Connection();
$db->connect();
$db->close();
```

Connect and Close are standard functions that will be called each time you want to start preparing
statements. It is recommended to keep the connection open across sequential statements and closed
once you are done with the db to keep the number of connections lower.

### The Database for these examples

**Table: my_table**

auto_incremented | some_int | text1 | text2 | time 
---------------- | -------- | ----- | ----- | ----
Empty |  |  |  | 

**Table: joining_table**

myid | firstname | lastname 
---- | --------- | -------- 
Empty | |   

### 1. Insert Statements

INSERT INTO table VALUES (?,?,?);

```php
$db = new Connection();
$db->connect();
$insert_id = $db->insert('my_table','iissi',array(null,1,'Some Text','key,other',time()));
$db->close();
```

**MySQL Statement**

```sql
INSERT INTO my_table VALUES(null, 1, 'Some Text', 'key,other', 1425338259);
```

This example opens the connection and performs a single insert and closes the connection. The insert is
to my_table with 5 parameters. The parameters are defined in the 3rd parameter as an array of parameters.
The table following the insert would look like:

auto_incremented | some_int | text1 | text2 | time 
---------------- | -------- | ----- | ----- | ----
1 | 1 | Some Text | key,other | 1425338259

Let's fill the tables now:


```php
$db = new Connection();
$db->connect();
$db->insert('my_table','iissi',array(null,2,'A longer description for me to search through','work,help',time()));
$db->insert('my_table','iissi',array(null,1,'I need some key words and common things for searching','tags,help,more',time()));
$db->insert('my_table','iissi',array(null,3,'Simulating a general key based tagging','key,work,more',time()));
$db->insert('joining_table','iss',array(1,'John','Stamos'));
$db->insert('joining_table','iss',array(2,'Steve','Banks'));
$db->insert('joining_table','iss',array(3,'Ella','Brooks'));
$db->insert('joining_table','iss',array(4,'Michael','Jones'));
$db->close();
```

and the resulting tables:

**Table: my_table**

auto_incremented | some_int | text1 | text2 | time 
---------------- | -------- | ----- | ----- | ----
1 | 1 | Some Text | key,other |  1425338259
2 | 2 | A longer description for me to search through | work,help |  1425338279
3 | 1 | I need some key words and common things for searching | tags,help,more |  1425338280
4 | 3 | Simulating a general key based tagging | key,work,more |  1425338281

**Table: joining_table**

myid | firstname | lastname 
---- | --------- | -------- 
1 | John | Stamos  
2 | Steve | Banks  
3 | Ella | Brooks 
4 | Michael | Jones

### 2. Update Statements

UPDATE table SET columns WHERE wherestatement

```php
$db = new Connection();
$db->connect();
$db->update('joining_table', 'si', array('firstname'), array('Sarah',1), 'where myid=?');
$db->close();
``` 

**MySQL Statement**

```sql
UPDATE joining_table SET firstname=? where myid=?;
```

Update statements probably make the least sense right away because of how they are structured. I say i am submitting
two variables, and yet only have one column: firstname. This is because the other variable is within the where statement.
Update statements have parameterized values in the SET part of the statement, and in the WHERE section. The columns then
specify which columns you are updating, and the prepared statements will handle placing things in the right place as
long as they are in the right order.

**Table: joining_table**

myid | firstname | lastname 
---- | --------- | -------- 
1 | Sarah | Stamos  
2 | Steve | Banks  
3 | Ella | Brooks 
4 | Michael | Jones

### 3. Select Statements

SELECT columns FROM table WHERE wherestatement

Select statements are the ones I used most with this class, and are often the most complex
statements you will have. The select statements I have ran within this function often push
what I thought was capable with this class. Think of the `columns`, `table`, and `wherestatement`
as placeholders for as much text as you would want in those areas, be it joins, group bys, 
IF, CASE, and so on.

#### Example 1

```php
$db = new Connection();
$db->connect();
$return = $db->select('my_table', array('*'),'s',array('search'),'where text1 like ? order by auto_incremented desc');
$db->close();
print_r($return);
``` 

**MySQL Query**
```sql
SELECT * FROM my_table where text1 like ? order by auto_incremented desc;
```

**Output**
```
Array(
  [0] => array( 
    'auto_incremented' => 3, 
	'some_int' => 1,
	'text1' => 'I need some key words and common things for searching',
	'text2' => 'tags,help,more',
	'time' => 1425338280
	),
  [1] => array( 
    'auto_incremented' => 2, 
	'some_int' => 2,
	'text1' => 'A longer description for me to search through',
	'text2' => 'work,help',
	'time' => 1425338279
	)	
);
```

I showed the actual output above, it's array of db rows in key => value format. From now on i'll just be
showing the nice markdown table of the output for my own sanity. As you can see there is very little required
in the wherestatement and it should reflect exactly what you would want normally in the where statement. This
can be cases for trimming down the return to the order, grouping, and limit. 

#### Example 2

```php
$db = new Connection();
$db->connect();
$return = $db->select(
    'my_table as a join joining_table as b on b.myid = a.auto_incremented', 
     array('a.auto_incremented','a.text2','b.firstname'),
	 's',
	 array('work'),
	 'where find_in_set(?,a.text2)>0'
	 );
$db->close();
``` 

**MySQL Query**
```sql
SELECT a.auto_incremented, a.text2, b.firstname FROM my_table as a join joining_table as b on b.myid = a.auto_incremented where find_in_set(?,a.text2)>0;
```

**The Return**

auto_incremented | text2 | firstname
---------------- | ----- | ---------
2 | work,help | Steve
4 | key,work,more | Michael

The functions more parse up certain aspects of a general sql query instead of forcing that parameter to just
be a table name. As such you can perform almost any query you want as long as you divide it up properly.
This statement does a joining of two tables where the join happens in the table parameter.

### 4. Delete Statements

DELETE FROM table wherestatement

Delete statements are always the scariest statements, and with the freedom I provide in these functions, they
still are the scariest statements.

#### Example 1

```php
$db = new Connection();
$db->connect();
$db->delete('joining_table', null, array(), '');
$db->close();
``` 

**MySQL Statement**
```sql
DELETE FROM joining_table;
```

**Output**

myid | firstname | lastname 
---- | --------- | -------- 
Empty | | 

The worst query to run on accident, the dump the whole table. With the function it requires some intention to
do so, but still entirely possible.