# Library Management System using SQL

## Project Overview

**Project Title:** Library Management System  
**Database:** `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

---

## Objectives

- **Set up the Library Management System Database:** Create and populate the database with tables for branches, employees, members, books, issued status, and return status.  
- **CRUD Operations:** Perform Create, Read, Update, and Delete operations on the data.  
- **CTAS (Create Table As Select):** Utilize CTAS to create new tables based on query results.  
- **Advanced SQL Queries:** Develop complex queries to analyze and retrieve specific data.  

---

## Project Structure

### 1. Database Setup

**Database Creation:**
```sql
CREATE DATABASE library_db;
Table Creation:
Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

sql
Copy code
-- Create table "Branch"
DROP TABLE IF EXISTS branch;
CREATE TABLE branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(15)
);

-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
    FOREIGN KEY (branch_id) REFERENCES branch(branch_id)
);

-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books (
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
CREATE TABLE issued_status (
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
CREATE TABLE return_status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(30),
    return_book_name VARCHAR(80),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);
2. CRUD Operations
Task 1: Create a New Book Record

sql
Copy code
INSERT INTO books(isbn, book_title, category, rental_price, status, author, publisher)
VALUES('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
SELECT * FROM books;
Task 2: Update an Existing Member's Address

sql
Copy code
UPDATE members
SET member_address = '125 Oak St'
WHERE member_id = 'C103';
Task 3: Delete a Record from the Issued Status Table

sql
Copy code
DELETE FROM issued_status
WHERE issued_id = 'IS121';
Task 4: Retrieve All Books Issued by a Specific Employee

sql
Copy code
SELECT * FROM issued_status
WHERE issued_emp_id = 'E101';
Task 5: List Members Who Have Issued More Than One Book

sql
Copy code
SELECT issued_emp_id, COUNT(*)
FROM issued_status
GROUP BY 1
HAVING COUNT(*) > 1;
3. CTAS (Create Table As Select)
Task 6: Create Summary Tables

sql
Copy code
CREATE TABLE book_issued_cnt AS
SELECT b.isbn, b.book_title, COUNT(ist.issued_id) AS issue_count
FROM issued_status AS ist
JOIN books AS b
ON ist.issued_book_isbn = b.isbn
GROUP BY b.isbn, b.book_title;
4. Data Analysis & Findings
Task 7: Retrieve All Books in a Specific Category

sql
Copy code
SELECT * FROM books
WHERE category = 'Classic';
Task 8: Find Total Rental Income by Category

sql
Copy code
SELECT b.category, SUM(b.rental_price), COUNT(*)
FROM issued_status AS ist
JOIN books AS b
ON b.isbn = ist.issued_book_isbn
GROUP BY 1;
List Members Who Registered in the Last 180 Days

sql
Copy code
SELECT * FROM members
WHERE reg_date >= CURRENT_DATE - INTERVAL '180 days';
List Employees with Their Branch Manager's Name

sql
Copy code
SELECT e1.emp_id, e1.emp_name, e1.position, e1.salary, b.*, e2.emp_name AS manager
FROM employees AS e1
JOIN branch AS b ON e1.branch_id = b.branch_id
JOIN employees AS e2 ON e2.emp_id = b.manager_id;
5. Advanced SQL Operations
Task 13: Identify Members with Overdue Books

sql
Copy code
SELECT 
    ist.issued_member_id,
    m.member_name,
    bk.book_title,
    ist.issued_date,
    CURRENT_DATE - ist.issued_date AS overdue_days
FROM issued_status AS ist
JOIN members AS m ON m.member_id = ist.issued_member_id
JOIN books AS bk ON bk.isbn = ist.issued_book_isbn
LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
WHERE rs.return_date IS NULL
AND (CURRENT_DATE - ist.issued_date) > 30
ORDER BY 1;
Task 14: Update Book Status on Return (Stored Procedure)

sql
Copy code
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$
DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
BEGIN
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT issued_book_isbn, issued_book_name INTO v_isbn, v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
END;
$$;
6. Reports & Analysis
Branch Performance Report

sql
Copy code
CREATE TABLE branch_reports AS
SELECT 
    b.branch_id,
    b.manager_id,
    COUNT(ist.issued_id) AS number_book_issued,
    COUNT(rs.return_id) AS number_book_return,
    SUM(bk.rental_price) AS total_revenue
FROM issued_status AS ist
JOIN employees AS e ON e.emp_id = ist.issued_emp_id
JOIN branch AS b ON e.branch_id = b.branch_id
LEFT JOIN return_status AS rs ON rs.issued_id = ist.issued_id
JOIN books AS bk ON ist.issued_book_isbn = bk.isbn
GROUP BY 1, 2;

SELECT * FROM branch_reports;
Conclusion
This project demonstrates the application of SQL skills in creating and managing a Library Management System. It includes database setup, data manipulation, and advanced querying — providing a solid foundation for database management and analytics.

How to Use
Clone the Repository

bash
Copy code
git clone https://github.com/AanchalSati/Library-Management-System.git
Set Up the Database
Execute the SQL scripts in the database_setup.sql file to create and populate the database.

Run the Queries
Use the SQL queries in the analysis_queries.sql file to perform data analysis.

Explore and Modify
Customize the queries to explore additional insights and relationships.




