# MySQL Connection Class

An easy class for handling prepared statements in PHP using mysqli. The class is fully commented
so here I won't explicitly tell what each function is for, just examples of how to use it in 
your own code.

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

Table: my_table

auto_incremented | some_int | text1 | text2 | time 
---------------- | -------- | ----- | ----- | ----
Empty |  |  |  | 

### 1. Insert Statements

```php
$db = new Connection();
$db->connect();
$insert_id = $db->insert('my_table','iissi',array(null,1,'Some Text','bye',time());
$db->close();
```

This example opens the connection and performs a single insert and closes the connection. The insert is
to my_table with 5 parameters. The parameters are defined in the 3rd parameter as an array of parameters.
The table following the insert would look like:

auto_incremented | some_int | text1 | text2 | time 
---------------- | -------- | ----- | ----- | ----
1 | 1 | Some Text | bye | 1425338259
