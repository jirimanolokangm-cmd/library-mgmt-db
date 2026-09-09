# Database Schema Documentation

## Table: Authors

**Description:** Stores information about book authors.

### Schema
```sql
CREATE TABLE Authors (
    author_id INT AUTO_INCREMENT PRIMARY KEY,
    author_name VARCHAR(100) NOT NULL,
    nationality VARCHAR(50)
);
```

### Columns
- **author_id**: Unique identifier for each author (auto-generated)
- **author_name**: Full name of the author (required)
- **nationality**: Country of origin for the author

### Relationships
- Referenced by: `Books.author_id` (One-to-Many)

### Sample Data
| author_id | author_name | nationality |
|-----------|-------------|-------------|
| 1 | Chinua Achebe | Nigerian |
| 2 | Ngugi wa Thiong'o | Kenyan |
| 3 | Chimamanda Ngozi Adichie | Nigerian |

---

## Table: Books

**Description:** Stores information about books available in the library with inventory tracking.

### Schema
```sql
CREATE TABLE Books (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(150) NOT NULL,
    author_id INT,
    category VARCHAR(50),
    published_year INT,
    quantity_available INT DEFAULT 1,
    FOREIGN KEY (author_id) REFERENCES Authors(author_id)
);
```

### Columns
- **book_id**: Unique identifier for each book (auto-generated)
- **title**: Title of the book (required)
- **author_id**: Foreign key reference to Authors table
- **category**: Genre or category of the book
- **published_year**: Year the book was published
- **quantity_available**: Number of copies available (default: 1)

### Relationships
- References: `Authors.author_id` (Many-to-One)
- Referenced by: `BorrowRecords.book_id` (One-to-Many)

### Sample Data
| book_id | title | author_id | category | published_year | quantity_available |
|---------|-------|-----------|----------|----------------|--------------------|
| 1 | Things Fall Apart | 1 | Fiction | 1958 | 5 |
| 2 | Weep Not, Child | 2 | Fiction | 1964 | 3 |
| 3 | Half of a Yellow Sun | 3 | History | 2006 | 4 |

---

## Table: Members

**Description:** Stores information about library members/patrons.

### Schema
```sql
CREATE TABLE Members (
    member_id INT AUTO_INCREMENT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE,
    phone VARCHAR(20),
    join_date DATE
);
```

### Columns
- **member_id**: Unique identifier for each member (auto-generated)
- **full_name**: Full name of the member (required)
- **email**: Email address of the member (unique)
- **phone**: Phone number for contact
- **join_date**: Date when the member joined the library

### Relationships
- Referenced by: `BorrowRecords.member_id` (One-to-Many)

### Sample Data
| member_id | full_name | email | phone | join_date |
|-----------|-----------|-------|-------|-----------|
| 1 | Jirimano Lokangm | jirimano@example.com | 0700000000 | 2026-09-01 |
| 2 | John Doe | john@example.com | 0711000000 | 2026-09-02 |

---

## Table: BorrowRecords

**Description:** Tracks all book borrowing and returning transactions.

### Schema
```sql
CREATE TABLE BorrowRecords (
    record_id INT AUTO_INCREMENT PRIMARY KEY,
    book_id INT,
    member_id INT,
    borrow_date DATE NOT NULL,
    return_date DATE,
    status VARCHAR(20) DEFAULT 'borrowed',
    FOREIGN KEY (book_id) REFERENCES Books(book_id),
    FOREIGN KEY (member_id) REFERENCES Members(member_id)
);
```

### Columns
- **record_id**: Unique identifier for each borrowing record (auto-generated)
- **book_id**: Foreign key reference to Books table (required)
- **member_id**: Foreign key reference to Members table
- **borrow_date**: Date when the book was borrowed (required)
- **return_date**: Date when the book was returned (NULL if not yet returned)
- **status**: Current status of the record (default: 'borrowed')
  - Values: `'borrowed'`, `'returned'`

### Relationships
- References: `Books.book_id` (Many-to-One)
- References: `Members.member_id` (Many-to-One)

### Sample Data
| record_id | book_id | member_id | borrow_date | return_date | status |
|-----------|---------|-----------|-------------|-------------|--------|
| 1 | 1 | 1 | 2026-09-09 | NULL | borrowed |
| 2 | 2 | 2 | 2026-09-08 | 2026-09-09 | returned |

---

## Data Integrity Constraints

### Primary Keys
- Ensure each table has a unique identifier
- Automatically generated using AUTO_INCREMENT

### Foreign Keys
- `Books.author_id` → `Authors.author_id`
- `BorrowRecords.book_id` → `Books.book_id`
- `BorrowRecords.member_id` → `Members.member_id`

### Unique Constraints
- `Members.email` - No two members can have the same email

### NOT NULL Constraints
- `Authors.author_name` - Must have author name
- `Books.title` - Must have book title
- `Members.full_name` - Must have member name
- `BorrowRecords.borrow_date` - Must record when book was borrowed

### Default Values
- `Books.quantity_available` - Defaults to 1
- `BorrowRecords.status` - Defaults to 'borrowed'

---

## Entity Relationship Diagram (ERD)

```
┌──────────┐
│ Authors  │
├──────────┤
│ author_id│ PK
│ name     │
│ country  │
└─────┬────┘
      │ 1:N
      │
      ▼
┌──────────────────┐         ┌─────────────────┐
│ Books            │◄────────┤ BorrowRecords   │
├──────────────────┤ 1:N  M:1├─────────────────┤
│ book_id          │ PK      │ record_id       │ PK
│ title            │         │ book_id         │ FK
│ author_id        │ FK      │ member_id       │ FK
│ category         │         │ borrow_date     │
│ published_year   │         │ return_date     │
│ quantity_available          │ status          │
└──────────────────┘         └────────┬────────┘
                                     │
                             1:N ◄───┤
                                     │
                            ┌────────▼────────┐
                            │ Members         │
                            ├─────────────────┤
                            │ member_id       │ PK
                            │ full_name       │
                            │ email           │ UNIQUE
                            │ phone           │
                            │ join_date       │
                            └─────────────────┘
```

---

## Notes

- All date fields use the DATE format (YYYY-MM-DD)
- Email addresses are unique across all members to prevent duplicates
- The BorrowRecords table maintains complete history of all transactions
- quantity_available tracks physical inventory
- The status field can be extended to include more values like 'overdue', 'reserved'

