CREATE DATABASE LibraryDB;
USE LibraryDB;
CREATE TABLE Books (
    BookID INT PRIMARY KEY IDENTITY(1,1),
    Title NVARCHAR(100),
    Author NVARCHAR(100),
    Genre NVARCHAR(50),
    IsAvailable BIT DEFAULT 1
);
CREATE TABLE Members (
    MemberID INT PRIMARY KEY IDENTITY(1,1),
    FullName NVARCHAR(100),
    JoinDate DATE DEFAULT GETDATE()
);
CREATE TABLE Borrowing (
    BorrowID INT PRIMARY KEY IDENTITY(1,1),
    BookID INT FOREIGN KEY REFERENCES Books(BookID),
    MemberID INT FOREIGN KEY REFERENCES Members(MemberID),
    BorrowDate DATE DEFAULT GETDATE(),
    DueDate DATE,
    Returned BIT DEFAULT 0
);
INSERT INTO Books (Title, Author, Genre)
VALUES ('1984', 'George Orwell', 'Dystopian'),
       ('The Catcher in the Rye', 'J.D. Salinger', 'Fiction'),
       ('To Kill a Mockingbird', 'Harper Lee', 'Classic');
INSERT INTO Members (FullName)
VALUES ('Alice Johnson'), ('Bob Smith'), ('Charlie Brown');
INSERT INTO Borrowing (BookID, MemberID, DueDate)
VALUES (1, 1, DATEADD(DAY, 14, GETDATE())),
       (2, 2, DATEADD(DAY, 14, GETDATE()));
SELECT B.Title, M.FullName, Bo.BorrowDate, Bo.DueDate
FROM Borrowing Bo
JOIN Books B ON Bo.BookID = B.BookID
JOIN Members M ON Bo.MemberID = M.MemberID
WHERE Bo.Returned = 0;
SELECT B.Title, M.FullName, Bo.DueDate
FROM Borrowing Bo
JOIN Books B ON Bo.BookID = B.BookID
JOIN Members M ON Bo.MemberID = M.MemberID
WHERE Bo.DueDate < GETDATE() AND Bo.Returned = 0;
SELECT * FROM Books WHERE IsAvailable = 1;
SELECT B.Title, DATEDIFF(DAY, Bo.DueDate, GETDATE()) * 10 AS Fine
FROM Borrowing Bo
JOIN Books B ON Bo.BookID = B.BookID
WHERE Bo.DueDate < GETDATE() AND Bo.Returned = 0;


CREATE PROCEDURE BorrowBook
    @BookID INT,
    @MemberID INT
AS
BEGIN
    IF EXISTS (SELECT 1 FROM Books WHERE BookID = @BookID AND IsAvailable = 1)
    BEGIN
        INSERT INTO Borrowing (BookID, MemberID, DueDate)
        VALUES (@BookID, @MemberID, DATEADD(DAY, 14, GETDATE()));
        UPDATE Books SET IsAvailable = 0 WHERE BookID = @BookID;
    END
    ELSE
    BEGIN
        PRINT 'Book is not available';
    END
END;
