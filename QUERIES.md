# SQL Queries Documentation

## Basic SELECT Queries

### 1. View All Books
```sql
SELECT * FROM Books;
```

### 2. View All Members
```sql
SELECT * FROM Members;
```

### 3. View All Authors
```sql
SELECT * FROM Authors;
```

### 4. View All Borrowing Records
```sql
SELECT * FROM BorrowRecords;
```

---

## Intermediate Queries (JOINs)

### 5. Get Books with Author Information
```sql
SELECT 
    b.book_id,
    b.title,
    a.author_name,
    b.category,
    b.published_year,
    b.quantity_available
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
ORDER BY b.title;
```

### 6. Get All Currently Borrowed Books
```sql
SELECT 
    b.title,
    m.full_name,
    br.borrow_date,
    b.category
FROM BorrowRecords br
JOIN Books b ON br.book_id = b.book_id
JOIN Members m ON br.member_id = m.member_id
WHERE br.status = 'borrowed';
```

### 7. Get All Returned Books
```sql
SELECT 
    b.title,
    m.full_name,
    br.borrow_date,
    br.return_date,
    DATEDIFF(br.return_date, br.borrow_date) AS days_borrowed
FROM BorrowRecords br
JOIN Books b ON br.book_id = b.book_id
JOIN Members m ON br.member_id = m.member_id
WHERE br.status = 'returned'
ORDER BY br.return_date DESC;
```

### 8. Get Complete Member Borrowing History
```sql
SELECT 
    m.full_name,
    b.title,
    a.author_name,
    br.borrow_date,
    br.return_date,
    br.status
FROM BorrowRecords br
JOIN Members m ON br.member_id = m.member_id
JOIN Books b ON br.book_id = b.book_id
JOIN Authors a ON b.author_id = a.author_id
WHERE m.member_id = 1
ORDER BY br.borrow_date DESC;
```

### 9. Get Books by Specific Author
```sql
SELECT 
    b.book_id,
    b.title,
    b.category,
    b.published_year,
    b.quantity_available
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
WHERE a.author_name = 'Chinua Achebe'
ORDER BY b.published_year;
```

### 10. Get Member Details with Borrowed Books
```sql
SELECT 
    m.member_id,
    m.full_name,
    m.email,
    b.title,
    br.borrow_date
FROM Members m
LEFT JOIN BorrowRecords br ON m.member_id = br.member_id
LEFT JOIN Books b ON br.book_id = b.book_id
WHERE br.status = 'borrowed'
ORDER BY m.full_name;
```

---

## Aggregation & Analytical Queries

### 11. Count Books by Category
```sql
SELECT 
    category,
    COUNT(*) AS total_books,
    SUM(quantity_available) AS available_copies
FROM Books
GROUP BY category
ORDER BY total_books DESC;
```

### 12. Get Books with Availability Status
```sql
SELECT 
    b.title,
    a.author_name,
    b.quantity_available,
    CASE 
        WHEN b.quantity_available > 0 THEN 'Available'
        ELSE 'Out of Stock'
    END AS status
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
ORDER BY b.quantity_available DESC;
```

### 13. Count Borrowing Activity by Member
```sql
SELECT 
    m.member_id,
    m.full_name,
    COUNT(br.record_id) AS total_borrows,
    SUM(CASE WHEN br.status = 'borrowed' THEN 1 ELSE 0 END) AS currently_borrowed,
    SUM(CASE WHEN br.status = 'returned' THEN 1 ELSE 0 END) AS returned
FROM Members m
LEFT JOIN BorrowRecords br ON m.member_id = br.member_id
GROUP BY m.member_id, m.full_name
ORDER BY total_borrows DESC;
```

### 14. Most Popular Books
```sql
SELECT 
    b.book_id,
    b.title,
    a.author_name,
    COUNT(br.record_id) AS borrow_count
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
LEFT JOIN BorrowRecords br ON b.book_id = br.book_id
GROUP BY b.book_id, b.title, a.author_name
ORDER BY borrow_count DESC;
```

### 15. Average Days a Book is Borrowed
```sql
SELECT 
    b.title,
    AVG(DATEDIFF(br.return_date, br.borrow_date)) AS avg_days_borrowed,
    COUNT(br.record_id) AS total_times_borrowed
FROM Books b
LEFT JOIN BorrowRecords br ON b.book_id = br.book_id
WHERE br.return_date IS NOT NULL
GROUP BY b.book_id, b.title
ORDER BY avg_days_borrowed DESC;
```

---

## Update & Maintenance Queries

### 16. Mark a Book as Returned
```sql
UPDATE BorrowRecords
SET return_date = CURDATE(), status = 'returned'
WHERE record_id = 1;
```

### 17. Increase Book Quantity
```sql
UPDATE Books
SET quantity_available = quantity_available + 2
WHERE book_id = 1;
```

### 18. Update Member Phone Number
```sql
UPDATE Members
SET phone = '0755555555'
WHERE member_id = 1;
```

### 19. Reduce Book Quantity When Borrowed
```sql
UPDATE Books
SET quantity_available = quantity_available - 1
WHERE book_id = 1;
```

---

## Deletion Queries (Use with Caution)

### 20. Delete a Borrowing Record
```sql
DELETE FROM BorrowRecords
WHERE record_id = 1;
```

### 21. Delete a Member (if no active borrowings)
```sql
DELETE FROM Members
WHERE member_id = 2;
```

### 22. Delete a Book (if no borrow records)
```sql
DELETE FROM Books
WHERE book_id = 3;
```

---

## Advanced Queries

### 23. Find Overdue Books (assuming 14 day limit)
```sql
SELECT 
    b.title,
    m.full_name,
    br.borrow_date,
    DATEDIFF(CURDATE(), br.borrow_date) AS days_borrowed
FROM BorrowRecords br
JOIN Books b ON br.book_id = b.book_id
JOIN Members m ON br.member_id = m.member_id
WHERE br.status = 'borrowed'
AND DATEDIFF(CURDATE(), br.borrow_date) > 14
ORDER BY days_borrowed DESC;
```

### 24. Get Recently Added Books
```sql
SELECT 
    b.book_id,
    b.title,
    a.author_name,
    b.published_year,
    b.quantity_available
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
ORDER BY b.book_id DESC
LIMIT 5;
```

### 25. Books by African Authors
```sql
SELECT 
    b.book_id,
    b.title,
    a.author_name,
    a.nationality,
    b.category,
    b.quantity_available
FROM Books b
JOIN Authors a ON b.author_id = a.author_id
WHERE a.nationality IN ('Nigerian', 'Kenyan')
ORDER BY a.nationality, b.title;
```

---

## Query Tips

1. **Use JOINs** to retrieve related data from multiple tables
2. **Use WHERE clauses** to filter results
3. **Use ORDER BY** to sort results
4. **Use GROUP BY** with aggregate functions (COUNT, SUM, AVG)
5. **Use LEFT JOIN** when you want to include records even if there's no match
6. **Use CASE statements** for conditional logic
7. **Always verify** before running UPDATE or DELETE queries
8. **Use LIMIT** to restrict number of results when testing

---

## Performance Considerations

- Add indexes on frequently searched columns (email, title)
- Use DATE functions instead of string comparisons for dates
- Avoid using wildcards at the beginning of LIKE patterns
- Use appropriate data types (INT for counts, DATE for dates)
- Consider denormalizing for frequently run reports

