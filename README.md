# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/najirh/Library-System-Management---P2/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/najirh/Library-System-Management---P2/blob/main/library_erd.png)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1. Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
insert into books (isbn, book_title, category, rental_price, status, author, publisher)
values 
('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
select * from books;
```
**Task 2: Update an Existing Member's Address**

```sql
update members
set member_address = '125 Main St'
where member_id = 'C101';

select * from members;

```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
delete from issued_status
where issued_id='IS121';

select * from issued_status;
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.
```sql
select * 
from 
	issued_status 
where 
	issued_emp_id = 'E101';
```


**Task 5: List Members Who Have Issued More Than One Book**
-- Objective: Use GROUP BY to find members who have issued more than one book.

```sql
select
	m.member_name, 
	count(i.issued_id) as no_of_books, 
	i.issued_member_id as member_id 
from 
	issued_status as i 
join 
	members as m 
on 
	m.member_id = i.issued_member_id 
group by 
	i.issued_member_id, m.member_name 
having 
	count(i.issued_id)>1;
```

### 3. CTAS (Create Table As Select)

- **Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt**

```sql
CREATE TABLE books_cnts AS
select 
	b.isbn,
	b.book_title as book_title,
	count(issued_book_isbn) as total_book_issued
from books as b
join issued_status as iss
on b.isbn=iss.issued_book_isbn
group by b.isbn, b.book_title

select * from books_cnts;
```


### 4. Data Analysis & Findings

The following SQL queries were used to address specific questions:

Task 7. **Retrieve All Books in a Specific Category**:

```sql
select book_title,
	category 
from 
	books 
where 
	category='Fiction';

```

8. **Task 8: Find Total Rental Income by Category**:

```sql
select sum(b.rental_price) as total_rental_income, category
from books as b
join issued_status as iss
on b.isbn = iss.issued_book_isbn
group by b.category;
```

9. **List Members Who Registered in the Last 180 Days**:
```sql
SELECT *
FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
```

10. **List Employees with Their Branch Manager's Name and their branch details**:

```sql
select 
	e1.*, 
	e2.emp_name as manager,
	b.manager_id
from employees e1
join branch b
on e1.branch_id=b.branch_id
join employees as e2
on b.manager_id = e2.emp_id

```

Task 11. **Create a Table of Books with Rental Price Above a Certain Threshold**:
```sql
create table book_rental_above_seven
as
select * 
from books 
where rental_price>7;
```

Task 12: **Retrieve the List of Books Not Yet Returned**
```sql
select distinct(issued_book_name) 
from issued_status as iss
left join return_status as ret
on iss.issued_id = ret.issued_id
where ret.return_id is NULL;
```

## Advanced SQL Operations

**Task 13: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
select iss.issued_member_id, 
	m.member_name, 
	b.book_title, 
	iss.issued_date, 
	current_date - iss.issued_date as over_due_days
from members m
join issued_status iss
on m.member_id= iss.issued_member_id
join books b
on b.isbn= iss.issued_book_isbn
left join return_status ret
on iss.issued_id= ret.issued_id
where ret.return_id is NULL and (current_date - iss.issued_date )>30
order by iss.issued_member_id
```


**Task 14: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

create or replace procedure add_return_records(p_return_id varchar(10),p_issued_id varchar(10), p_book_quality varchar(10))
language plpgsql
as $$

declare 
	v_isbn varchar(50);
	v_book_name varchar(80);
	
begin
	insert into return_status(return_id,issued_id,return_date, book_quality) 
	values
	(p_return_id, p_issued_id, current_date, p_book_quality);

	select
		issued_book_isbn,
		issued_book_name
		into
		v_isbn,
		v_book_name
	from issued_status
	where issued_id=p_issued_id;

	update books
	set status='yes'
	where isbn=v_isbn;

	raise notice 'thank you for returning the book: %',v_book_name;

end;
$$

--calling function
select * from issued_status;
select * from return_status;
call add_return_records('RS138','IS135','Good');

```




**Task 15: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
select * from branch;
select * from books;
select * from employees;
select * from return_status;

create table 
	branch_reports 
as 
select
	br.branch_id, 
	br.manager_id, 
	count(iss.issued_id) as total_issued_books, 
	count(rs.return_id) as total_books_returned, 
	sum(bk.rental_price) as total_revenue
from issued_status iss
join employees emp
on iss.issued_emp_id=emp.emp_id
join branch br
on emp.branch_id=br.branch_id
left join return_status as rs
on rs.issued_id=iss.issued_id
join books bk
on iss.issued_book_isbn=bk.isbn
group by 
	br.branch_id, 
	br.manager_id
order by 
	br.branch_id;

select * from branch_reports;
```

**Task 16: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql

create table active_members as
select m.member_id, 
	m.member_name, 
	m.member_address, 
	m.reg_date 
from 
	members m
join 
	issued_status iss
on
	m.member_id=iss.issued_member_id
where iss.issued_date >= current_date - interval '2 month'

select * from active_members;

```


**Task 17: Find Employees with the Most Book Issues Processed**  
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
select 
	e.emp_id,
	e.emp_name, 
	count(iss.issued_id) as number_of_books_issued, 
	e.branch_id 
from 
	employees as e
join 
	issued_status as iss
on 
	e.emp_id=iss.issued_emp_id
group by e.emp_id
order by count(iss.issued_id) desc
limit 3;
```


**Task 18: Stored Procedure**
Objective:
Create a stored procedure to manage the status of books in a library system.
Description:
Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql

create or replace procedure issue_book(p_issued_id varchar(10),p_issued_member_id varchar(30),p_issued_book_isbn varchar(30) , p_issued_emp_id varchar(10))
language plpgsql
as $$

declare
	v_status varchar(10);

begin
	select status
	into v_status 
	from books 
	where isbn=p_issued_book_isbn;

	if v_status = 'yes' then
		insert into issued_status (issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
		values 
		(p_issued_id, p_issued_member_id, current_date, p_issued_book_isbn, p_issued_emp_id);

		update books
		set status='no'
		where isbn= p_issued_book_isbn;

		raise notice 'Book records added successfully for book isbn %', p_issued_book_isbn;

	else
		raise notice 'Sorry to inform you the book you have asked is unavailable at the moment';
	end if;

end;
$$

-- calling procedure
CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');

CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

```







## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.



