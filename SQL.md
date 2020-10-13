# Joins:
[REf](https://www.youtube.com/watch?v=KTvYHEntvn8 )
	The SQL Joins clause is used to combine records from two or more tables in a database. A JOIN is a means for combining fields from two tables by using values common to each.

## Inner Joins/EQUIJOIN:

	The query compares each row of table1 with each row of table2 to find all pairs of rows which satisfy the join-predicate. When the join-predicate is satisfied, column values for each matched pair of rows of A and B are combined into a result row.
eg:
	select * from tblCountry 
		INNER JOIN tblState
		ON tblCountry.Countryid = tblState.Countryid

Results only matching records.

## Left JOIN
	The SQL LEFT JOIN returns all rows from the left table, even if there are no matches in the right table.
eg:
	select * from tblCountry 
		LEFT JOIN tblState
		ON tblCountry.Countryid = tblState.Countryid
	
	
## Right Join
		Returns all rows from the right table, even if there are no matches in the left table.
eg:-		
		select * from tblCountry 
				RIGHT JOIN tblState
				ON tblCountry.Countryid = tblState.Countryid
				
## FullOuter Join				
	The SQL FULL JOIN combines the results of both left and right outer joins.
eg:-
select * from tblCountry 
				FULL OUTER JOIN tblState
				ON tblCountry.Countryid = tblState.Countryid
	
## Self JOIN
		Joining a Table with Itself.
	eg:-find the employee details who are getting more salary than the manager's salary.
	
***************************************************************************************************
**Q) What is a Primary key?**
	
	Primary key is a column (or collection of columns) or a set of columns that uniquely identifies each row in the table.
	Uniquely identifies a single row in the table
	Null values not allowed

**Q) What is Foreign Key **
	
	Foreign key maintains referential integrity by enforcing a link between the data in two tables.
	The foreign key in the child table references the primary key in the parent table.
	The foreign key constraint prevents actions that would destroy links between the child and parent tables.
	
	
	
**Q) What is the difference between DELETE and TRUNCATE statements?**	
	
	DELETE:
	Delete command is used to delete a row in a table.
	You can rollback data after using delete statement.
	It is a DML command.
	It is slower than truncate statement.
	
	Trucate:
	Truncate is used to delete all the rows from a table.
	You cannot rollback data.
	It is a DDL command.
	It is faster.

**Q) What are Constraints in SQL**

	Constraints are the rules enforced on the data columns of a table
	
	NOT NULL Constraint − Ensures that a column cannot have NULL value.
	DEFAULT Constraint − Provides a default value for a column when none is specified.
	UNIQUE Constraint − Ensures that all values in a column are different.
	PRIMARY Key − Uniquely identifies each row/record in a database table.
	FOREIGN Key − Uniquely identifies a row/record in any of the given database table.
	CHECK Constraint − The CHECK constraint ensures that all the values in a column satisfies certain conditions.
	INDEX − Used to create and retrieve data from the database very quickly.

**Q). What is the difference between CHAR and VARCHAR2 datatype in SQL?**

	Both Char and Varchar2 are used for characters datatype but varchar2 is used for character strings of variable length whereas Char is used for strings of fixed length.
	
**Q) What is Stored Procedure in SQL?**
	
	It is nothing but a group of sql statements that accepts some inputs in form of parameters and perform some tasks and may or may not return some value.
	eg:-
	Imagine a table named with emp_table stored in Database. We are Writing a Procedure to update a Salary of Employee with 1000.
	
```sql
	CREATE OR REPLACE PROCEDURE INC_SAL(eno IN NUMBER, up_sal OUT NUMBER)
	IS
	BEGIN
	UPDATE 	emp_table SET SALARY = SALARY+1000 WHERE emp_no  = eno;
	COMMIT;
	SELECT sal INTO up_sal FROM emp_table WHERE emp_no = eno;
	END;
```
	Steps to execute the procedure:
	Declare a Variable to Store the value comming out from Procedure :
	VARIABLE v NUMBER;
	
	Execution of the Procedure
	EXECUTE INC_SAL(1002, :v);
	
	To check the updated salary use SELECT statement
	SELECT all(star) FROM emp_table WHERE emp_no = 1002;
	
**Q) Discuss Index In sql	**
	Indexing is used to efficiently retrieve records from the db, based on the attribute on which indexing has been done.
	It is used by the server to speed up the retrieval of rows by using a pointer.
	How you do that ?
	By The CREATE INDEX Command.
	CREATE INDEX index_name ON table_name;
	
	Types of Indexes:
	1) Single-Column Indexes:- A single-column index is created based on only one table column.
		CREATE INDEX index_name
		ON table_name (column_name);
	
	2) 	Unique Indexes:
	 A unique index does not allow any duplicate values to be inserted into the table. The basic syntax is as follows.
	 	CREATE UNIQUE INDEX index_name
		on table_name (column_name);
	
	3) Composite Indexes:
	 A Composite Indexes is a index  on two or more columns of a table.
		CREATE INDEX index_name
		on table_name(Column1, Column2);
		
	Drop Index:
		An index can be dropped using SQL DROP command.
		DROP INDEX index_name;
		
	The following guidelines indicate when the use of an index should be reconsidered.

	Indexes should not be used on small tables.
	Tables that have frequent, large batch updates or insert operations.
	Indexes should not be used on columns that contain a high number of NULL values.
	Columns that are frequently manipulated should not be indexed.
		
	Types of Indexes:-
	Unique Index: This index does not allow the field to have duplicate values if the column is unique indexed. If a primary key is defined, a unique index can be applied automatically.
	Clustered Index: This index reorders the **physical order** of the table and searches based on the basis of key values. Each table can only have one clustered index.
	Non-Clustered Index: Non-Clustered Index does not alter the physical order of the table and maintains a **logical order** of the data. Each table can have many nonclustered indexes.
	
	
**Q) What is the difference between clustered and non clustered index in SQL?**
	
	Clustered Index:			
	Its created on primary key.
	Store data physically according to the order.
	Only one clustered index can be there in a table.
	No extra space is required to store logical structure.
	Data retrieval is faster than non-cluster index 

	Non-Clustered Index:
	It can be created on any key.
	It don’t impact the order.
	There can be any number of non-clustered indexes in a table.
	Extra space is required to store logical structure.
	Data update is faster than clustered index.
	
				
**Q)	Difference between FetchType LAZY and EAGER in Java Persistence API?**

		 The FetchType defines when Hibernate gets the related entities from the database.
		 To load it together with the rest of the fields (i.e. eagerly), -- Eager loading
		 To load it on-demand (i.e. lazily) when you call the university's getStudents() method.
		 LAZY = fetch when needed
		 EAGER = fetch immediately
		 
**Q)	@DirtiesContext in Unit Test?**
		 To Clear the Dirt which is created in unit Test like Update or insert.
		 To leave the data in consistent state.
		 
		 
*******************************************************

**Q Sql Query to find highest salary?**
	Select Max(salary) from emp;
	or 
	select salary from emp orderby salary desc limit 1;

**Q Sql Query to find 2nd highest salary?**
	
	Select Max(sal) from emp 
	where sal< (Select Max(salary) from emp);
		
**Q Sql Query to find nth highest salary?**
	
	select * from(
	select ename, sal, dense_rank() 
	over(order by sal desc)r from Employee) 
	where r=&n;