# Library Management System - PostgreSQL

## Description

A comprehensive library management system database built with PostgreSQL. This project demonstrates relational database design, SQL queries, and data manipulation for managing books, authors, members, and borrowing records.

## Features

- **Complete Database Schema**: 4 interconnected tables with proper foreign key relationships
- **Rich Dataset**: Pre-populated with 18 famous authors, 21 classic books, 18 members, and 30+ borrowing records
- **19 SQL Queries**: From basic to advanced queries covering real-world library scenarios
- **Data Integrity**: Proper constraints and relationships to maintain data consistency

## Database Structure

### Tables

1. **Authors**
   - AuthorID (Primary Key)
   - FirstName
   - LastName

2. **Books**
   - BookID (Primary Key)
   - Title
   - AuthorID (Foreign Key → Authors)
   - Genre
   - PublishedYear
   - Quantity

3. **Members**
   - MemberID (Primary Key)
   - FirstName
   - LastName
   - MembershipDate

4. **BorrowingRecords**
   - RecordID (Primary Key)
   - MemberID (Foreign Key → Members)
   - BookID (Foreign Key → Books)
   - BorrowDate
   - ReturnDate

## Installation

### Prerequisites

- PostgreSQL 12 or higher
- psql command-line tool or any PostgreSQL client (pgAdmin, DBeaver, etc.)

### Setup

1. Clone the repository:
```bash
git clone https://github.com/malijama/library-management-postgresql.git
cd library-management-postgresql
```

2. Create a database:
```sql
CREATE DATABASE library_db;
```

3. Run the SQL script:
```bash
psql -d library_db -f library_management_system_complete.sql
```

## Sample Data

### Authors Include
- J.K. Rowling, George Orwell, J.R.R. Tolkien
- Isaac Asimov, Agatha Christie, Mark Twain
- F. Scott Fitzgerald, Jane Austen
- And 10 more classic authors

### Books Include
- Harry Potter series, 1984, The Hobbit
- Foundation, Murder on the Orient Express
- The Great Gatsby, Pride and Prejudice
- And 14 more classic titles

## Query Examples

### Basic Queries

**Q1: Find all books borrowed by John**
```sql
SELECT firstname || ' ' || lastname AS fullname, title
FROM books b
JOIN borrowingrecords b2 ON b.bookid = b2.bookid
JOIN members m ON b2.memberid = m.memberid
WHERE m.firstname LIKE 'John%';
```

**Q2: List all available books (not borrowed)**
```sql
SELECT title
FROM books b
WHERE b.bookid NOT IN (
    SELECT b2.bookid FROM borrowingrecords b2
);
```

**Q3: Count total books by each author**
```sql
SELECT COUNT(title) AS number_of_books,
       a.firstname || ' ' || a.lastname AS authors_name
FROM books b
JOIN authors a ON b.authorid = a.authorid
GROUP BY a.firstname, a.lastname
ORDER BY number_of_books;
```

### Intermediate Queries

**Q7: Authors with more than one book**
```sql
SELECT a.firstname || ' ' || a.lastname AS authors_name,
       COUNT(*) AS number_of_books
FROM authors a
JOIN books b ON a.authorid = b.authorid
GROUP BY authors_name
HAVING COUNT(*) > 1
ORDER BY number_of_books DESC;
```

**Q8: Average books borrowed per member**
```sql
SELECT ROUND(AVG(books_count), 2) AS average
FROM (
    SELECT m.firstname || ' ' || m.lastname AS members_name,
           COUNT(*) AS books_count
    FROM members m
    JOIN borrowingrecords b ON m.memberid = b.memberid
    GROUP BY members_name
);
```

### Advanced Queries

**Q18: Find the most borrowed book (using window functions)**
```sql
SELECT title, most_borrowed
FROM (
    SELECT b.title,
           COUNT(*) AS most_borrowed,
           DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS ranks
    FROM books b
    JOIN borrowingrecords b2 ON b.bookid = b2.bookid
    GROUP BY b.title
) ranked
WHERE ranks = 1;
```

**Q19: Member with most books borrowed**
```sql
SELECT member_name, number_books
FROM (
    SELECT firstname || ' ' || lastname AS member_name,
           COUNT(b2.memberid) AS number_books,
           DENSE_RANK() OVER (ORDER BY COUNT(*) DESC) AS ranks
    FROM members m
    JOIN borrowingrecords b2 ON m.memberid = b2.memberid
    GROUP BY firstname || ' ' || lastname
) ranked
WHERE ranks = 1;
```

## All 19 Queries Covered

1. Find all books borrowed by a specific member
2. List all available books (not borrowed)
3. Count total books by each author
4. List all books by genre
5. List members who haven't borrowed any books
6. Find total books available in the library
7. List authors with more than one book
8. Calculate average books borrowed per member
9. List books that have never been borrowed
10. Find books borrowed within a date range
11. Determine the most popular genre
12. Find overdue books (not returned in 30 days)
13. Find all currently borrowed books
14. Identify members with more than one book
15a. List overdue books with days overdue
15b. List books borrowed at least twice
16. Find difference in books borrowed between two members
17. Identify members who borrowed books by a specific author
18. Find the most borrowed book (advanced)
19. Identify member with most books borrowed (advanced)

## SQL Techniques Demonstrated

- **Joins**: INNER JOIN, LEFT JOIN, CROSS JOIN
- **Subqueries**: IN, NOT IN, nested subqueries
- **Aggregation**: COUNT, SUM, AVG with GROUP BY
- **String Operations**: Concatenation with ||
- **Date Functions**: CURRENT_DATE, date arithmetic
- **Window Functions**: DENSE_RANK() OVER()
- **Filtering**: WHERE, HAVING clauses
- **Distinct**: Remove duplicates
- **Sorting**: ORDER BY with ASC/DESC

## Use Cases

This database system can handle:
- Book inventory management
- Member registration and tracking
- Borrowing and return operations
- Overdue book identification
- Popular genre analysis
- Member activity tracking
- Author and book statistics

## Author

**Mohamed** ([@malijama](https://github.com/malijama))

## License

This project is open source and available for educational purposes.

---

**Perfect for**: Learning SQL, Database design practice, Portfolio projects, SQL interview preparation
