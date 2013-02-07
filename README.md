sp\_generate\_inserts
=====================

This tool is orriginally from http://vyaskn.tripod.com/code.htm#inserts. My attempts to get bug fixes accepted back have not resulted in any response so I have posted my altered version here.

Documentation
=============

This procedure generates INSERT statements using existing data from the given tables and views. Later, you can use these INSERT statements to generate the data. It's very useful when you have to ship or package a database application. This procedure also comes in handy when you have to send sample data to your vendor or technical support provider for troubleshooting purposes. 

Advantages
----------

* Data from both tables and views can be scripted
* No CURSORs are used
* Table names and column names with spaces are handled
* All datatypes are handled except images, large text and binary columns with more than 4 bytes
* NULLs are gracefully handled
* Timestamp columns are handled
* Identity columns are handled
* Very flexible and configurable
* Non-dbo owned tables are handled 
* Computed columns are handled
* You can filter the rows for which you want to generate INSERTs

Examples
--------

### Example 1     

To generate INSERT statements for table 'titles':

```sql
EXEC sp_generate_inserts 'titles'
```

### Example 2 

To ommit the column list in the INSERT statement: (Column list is included by default)
NOTE: If you have too many columns, you are advised to ommit column list, as shown below, to avoid erroneous results

```sql
EXEC sp_generate_inserts 'titles', @Include_Column_List = 0
```

### Example 3

To generate INSERT statements for 'titlesCopy' table from 'titles' table:

```sql
EXEC sp_generate_inserts 'titles', 'titlesCopy'
```

### Example 4

To generate INSERT statements for 'titles' table for only those titles which contain the word 'Computer' in them:

```sql
EXEC sp_generate_inserts 'titles', @From = "from titles where title like '%Computer%'"
```

### Example 5

To specify that you want to include TIMESTAMP column's data as well in the INSERT statement:
NOTE: By default TIMESTAMP column's data is not scripted

```sql
EXEC sp_generate_inserts 'titles', @Include_Timestamp = 1
```

### Example 6:

To print the debug information:

```sql
EXEC sp_generate_inserts 'titles', @debug_mode = 1
```

### Example 7

If you are not the owner of the table, use @owner parameter to specify the owner name:
NOTE: To use this option, you must have SELECT permissions on that table

```sql
EXEC sp_generate_inserts Nickstable, @owner = 'Nick'
```

###Example 8

To generate INSERT statements for the rest of the columns excluding images:
NOTE: When using this otion, DO NOT set @include_column_list parameter to 0

```sql
EXEC sp_generate_inserts imgtable, @ommit_images = 1
```

### Example 9

To generate INSERT statements for the rest of the columns excluding IDENTITY column:

```sql
EXEC sp_generate_inserts mytable, @ommit_identity = 1
```

### Example 10

To generate INSERT statements for the top 10 rows in the table:

```sql
EXEC sp_generate_inserts mytable, @top = 10
```

### Example 11

To generate INSERT statements only with the columns you want:

```sql
EXEC sp_generate_inserts titles, @cols_to_include = "'title','title_id','au_id'"
```

### Example 12

To generate INSERT statements by ommitting some columns:

```sql
EXEC sp_generate_inserts titles, @cols_to_exclude = "'title','title_id','au_id'"
```

### Example 13

To avoid checking the foreign key constraints while loading data with INSERT statements:
NOTE: The @disable_constraints option will disable foreign key constraints, by assuming that the source data is valid and referentially sound

```sql
EXEC sp_generate_inserts titles, @disable_constraints = 1
```

### Example 14

To avoid scripting data from computed columns:

```sql
EXEC sp_generate_inserts MyTable, @ommit_computed_cols = 1
```


NOTE: Please see the code and read the comments to understand more about how this procedure works!

Output All Tables in a Database
-------------------------------

To generate INSERT statements for all the tables in your database, execute the following query in that database, which will output the commands, that you need to execute for the same:

```sql
SELECT 'EXEC sp_generate_inserts ' + 
'[' + name + ']' + 
',@owner = ' + 
'[' + RTRIM(USER_NAME(uid)) + '],' + 
'@ommit_images = 1, @disable_constraints = 1'
FROM sysobjects 
WHERE type = 'U' AND 
OBJECTPROPERTY(id,'ismsshipped') = 0
```

Truncated Output
----------------

If output in Query Analyzer seems truncated and you can only see the first 255 characters per line you need to increase the line length limit using one of the folling steps:

If you are using the Query Analyzer of Server 2000, go to Tools -> Options -> Results tab -> Enter the number 8192 in the "Maximum characters per column:" text box.

If you are using the Query Analyzer of Server 7.0, go to Query -> Current Connection Options -> Advanced tab -> Enter the number 8192 in the "Maximum characters per column:" text box.
