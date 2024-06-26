# SENG8071 Midterm Assignment

## Table Design

### Books Table
|     Name      |      Type     |
| ------------- |---------------|
| book_id       | PRIMARY KEY   |
| title         | VARCHAR(255)  |
| author_name   | VARCHAR(255)  |
| publisher_name| VARCHAR(255)  |
| publish_date  | DATE          |
| price         | DECIMAL(10, 2)|
| format        | VARCHAR(50)   |
| rating        | DECIMAL(2, 1) |
| genre         | VARCHAR(100)  |

### Customers Table
|     Name      |      Type     |
| ------------- |---------------|
| customer_id   | PRIMARY KEY   |
| name          | VARCHAR(255)  |
| email         | VARCHAR(255)  |
| join_date     | DATE          |

### Purchases Table
|     Name      |      Type     |
| ------------- |---------------|
| purchase_id   | PRIMARY KEY   |
| customer_id   | INT           |
| book_id       | INT           |
| purchase_date | DATE          |
| total_price   | DECIMAL(10, 2)|

### Reviews Table
|     Name      |      Type     |
| ------------- |---------------|
| review_id     | PRIMARY KEY   |
| customer_id   | INT           |
| book_id       | INT           |
| review_date   | DATE          |
| rating        | INT           |
| review_text   | TEXT          |

## DDL & DML
### Create Table
```sql
-- Books Table
    CREATE TABLE Books (
        book_id SERIAL PRIMARY KEY,
        title VARCHAR(255) NOT NULL,
        author_name VARCHAR(255) NOT NULL,
        publisher_name VARCHAR(255) NOT NULL,
        publish_date DATE NOT NULL,
        price DECIMAL(10, 2) NOT NULL,
        format VARCHAR(50) NOT NULL CHECK (format IN ('Physical', 'E-book', 'Audiobook')),
        rating DECIMAL(2, 1) CHECK (rating >= 1.0 AND rating <= 5.0),
        genre VARCHAR(100) NOT NULL
    );

-- Customers Table
    CREATE TABLE Customers (
        customer_id SERIAL PRIMARY KEY,
        name VARCHAR(255) NOT NULL,
        email VARCHAR(255) UNIQUE NOT NULL,
        join_date DATE NOT NULL
    );

-- Purchases Table
    CREATE TABLE Purchases (
        purchase_id SERIAL PRIMARY KEY,
        customer_id INT REFERENCES Customers(customer_id),
        book_id INT REFERENCES Books(book_id),
        purchase_date DATE NOT NULL,
        total_price DECIMAL(10, 2) NOT NULL
    );

-- Reviews Table
    CREATE TABLE Reviews (
        review_id SERIAL PRIMARY KEY,
        customer_id INT REFERENCES Customers(customer_id),
        book_id INT REFERENCES Books(book_id),
        review_date DATE NOT NULL,
        rating INT CHECK (rating >= 1 AND rating <= 5),
        review_text TEXT
    );
```
### Insert mock data
```sql
    INSERT INTO Books (title, author_name, publisher_name, publish_date, price, format, rating, genre) VALUES
    ('Book1', 'Author1', 'Publisher1', '2024-04-05', 20.00, 'Physical', 4.6, 'Fiction'),
    ('Book2', 'Author2', 'Publisher2', '2024-06-07', 15.00, 'E-book', 3.5, 'Non-Fiction'),
    ('Book3', 'Author1', 'Publisher1', '2023-08-09', 25.00, 'Audiobook', 4.1, 'Fiction');

    INSERT INTO Customers (name, email, join_date) VALUES
    ('Customer1', 'customer1xxx@xxxxxx.com', '2011-02-04'),
    ('Customer2', 'customer2xxx@xxxxxx.com', '2012-03-05');

    INSERT INTO Purchases (customer_id, book_id, purchase_date, total_price) VALUES
    (1, 1, '2024-01-01', 20.00),
    (2, 2, '2024-02-01', 15.00),
    (1, 3, '2024-03-01', 25.00);

    INSERT INTO Reviews (customer_id, book_id, review_date, rating, review_text) VALUES
    (1, 1, '2024-01-15', 5, 'Good!'),
    (2, 2, '2024-02-15', 4, 'Not bad'),
    (1, 3, '2024-03-15', 5, 'Love it!');
```

## Clearly Identify 1 complete set of DML For one of the tables
```sql
-- Insert is above

-- Delete
    DELETE FROM Books
    WHERE book_id = 3;

-- Update
    UPDATE Books
    SET price = 22.00
    WHERE book_id = 1;

-- Query
    -- Query All
        SELECT * FROM Books;

    -- Query 1
        SELECT * FROM Books
        WHERE book_id = 1;
```

## The query sentences of user's requirements

### Power writers (authors) with more than X books in the same genre published within the last X years
```sql
    SELECT 
        author_name, 
        genre, 
        COUNT(book_id) AS book_count
    FROM Books
    WHERE publish_date >= CURRENT_DATE - INTERVAL 'X years'
    GROUP BY author_name, genre
    HAVING COUNT(book_id) > X;
```

### Loyal Customers who has spent more than X dollars in the last year
```sql
    SELECT 
        c.name, 
        c.email, 
        SUM(p.total_price) AS total_spent
    FROM Customers c
    JOIN Purchases p ON c.customer_id = p.customer_id
    WHERE p.purchase_date >= CURRENT_DATE - INTERVAL '1 year'
    GROUP BY c.name, c.email
    HAVING SUM(p.total_price) > X;
```

### Well Reviewed books that has a better user rating than average
```sql
    SELECT 
        title, 
        author_name, 
        publisher_name, 
        rating
    FROM Books
    WHERE rating > (SELECT AVG(rating) FROM Books);
```

### The most popular genre by sales
```sql
    SELECT 
        genre, 
        SUM(p.total_price) AS total_sales
    FROM Purchases p
    JOIN Books b ON p.book_id = b.book_id
    GROUP BY genre
    ORDER BY total_sales DESC
    LIMIT 1;
```

### The 10 most recent posted reviews by Customers 
```sql
    SELECT 
        c.name AS customer_name, 
        b.title AS book_title, 
        r.rating, 
        r.review_text, 
        r.review_date
    FROM Reviews r
    JOIN Customers c ON r.customer_id = c.customer_id
    JOIN Books b ON r.book_id = b.book_id
    ORDER BY r.review_date DESC
    LIMIT 10;
```
## Typescript
### Define an interface for the Books table
```
interface Book {
    book_id: number;
    title: string;
    author_name: string;
    publisher_name: string;
    publish_date: string;
    price: number;
    format: 'Physical' | 'E-book' | 'Audiobook';
    rating: number;
    genre: string;
}
```

### Define a generic interface for table modification
```
interface TableModification<T> {
    insert(item: T): Promise<void>;
    update(item: T): Promise<void>;
    delete(id: number): Promise<void>;
}
```

### Implement the interface for the Books table
```
class BooksTable implements TableModification<Book> {
    async insert(book: Book): Promise<void> {
        // Implementation for inserting a book into the database
        const query = INSERT INTO Books (title, author_name, publisher_name, publish_date, price, format, rating, genre) VALUES ($1, $2, $3, $4, $5, $6, $7, $8);
        const values = [book.title, book.author_name, book.publisher_name, book.publish_date, book.price, book.format, book.rating, book.genre];
        // Execute the query by using database client
        // await dbClient.query(query, values);
    }

    async update(book: Book): Promise<void> {
        if (!book.book_id) throw new Error('Book ID is required for update.');
        // Implementation for updating a book in the database
        const query = UPDATE Books SET title = $1, author_name = $2, publisher_name = $3, publish_date = $4, price = $5, format = $6, rating = $7, genre = $8 WHERE book_id = $9;
        const values = [book.title, book.author_name, book.publisher_name, book.publish_date, book.price, book.format, book.rating, book.genre, book.book_id];
        // Execute the query by using database client
        // await dbClient.query(query, values);
    }

    async delete(id: number): Promise<void> {
        // Implementation for deleting a book from the database
        const query = DELETE FROM Books WHERE book_id = $1;
        const values = [id];
        // Execute the query by using database client
        // await dbClient.query(query, values);
    }
}
```
