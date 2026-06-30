# 🗺️ Relational Schema Design: The Blueprint of Structured Data

When building a Relational Database Management System (RDBMS) like PostgreSQL or MySQL, your primary goal is to design a **Schema**—a structural blueprint that defines how data is organized, linked, and protected.

---

## 🏢 The Core Analogy: The Luxury Hotel Engine

Think of a Relational Database Schema as the **operational blueprint of a five-star luxury hotel**. 

To run a hotel efficiently, you don't throw guest names, room numbers, employee salaries, and dinner orders into one giant, messy spreadsheet. Instead, you create dedicated, structured systems that talk to each other safely.

---

## 🔑 Core Concepts Explained

### 1. Tables (The Dedicated Ledger Books)
In our hotel, you have different physical ledger books kept at the front desk:
* One ledger for **Guests** (storing names, emails, phone numbers).
* One ledger for **Rooms** (storing room numbers, bed types, prices).
* One ledger for **Bookings** (storing who checked into which room and when).

> **In a Database:** A **Table** is a collection of related data entries consisting of columns (attributes) and rows (records). You isolate distinct entities into their own tables to prevent messy data duplication.



### 2. Primary Key (The Unique Keycard / ID)
If you have three guests named "John Smith," how does the hotel know who is who? When a guest checks in, they are handed a **Unique Guest ID** or a specific keycard programmed with a one-of-a-kind serial number. 

* Even if details match, the ID never changes and is never reused.
* A room number (e.g., Room 404) is the Primary Key for the Rooms ledger.

> **In a Database:** A **Primary Key (PK)** is a column (or group of columns) that uniquely identifies each row in a table. It cannot be null, and it absolutely cannot contain duplicate values.

### 3. Foreign Key (The Booking Voucher)
When a guest reserves a room, the receptionist opens the **Bookings Ledger** and writes down:
`Booking #999 | Guest ID: 45 | Room Number: 404`

The `Guest ID` and `Room Number` written in the Bookings ledger point back directly to the official records in the Guests and Rooms ledgers. 

> **In a Database:** A **Foreign Key (FK)** is a column in one table that points directly to the Primary Key of another table. This establishes the **relationship** between data sets.

### 4. Constraints (The Hotel House Rules)
Constraints are the automated guardrails that prevent human error from corrupting the system.

* **NOT NULL Constraint:** You cannot check a guest in without a valid Phone Number or First Name. The field cannot be left blank.
* **UNIQUE Constraint:** No two rooms can have the exact same Room Number. No two guests can register under the exact same Email Address.
* **CHECK Constraint:** The room price column must always be greater than $0. (You can't accidentally pay a guest to stay there).
* **FOREIGN KEY / Referential Integrity Constraint:** You cannot write a booking for "Room 999" if the hotel only has rooms up to 500. The database will reject the entry because Room 999 does not exist in the Rooms ledger.

---

## 📊 Visual Schema Map

Here is how these pieces connect conceptually inside the system:

| Table: `GUESTS` | | Table: `BOOKINGS` | | Table: `ROOMS` |
| :--- | :--- | :--- | :--- | :--- |
| **guest_id** (PK)<br>first_name<br>email (UNIQUE) | &larr; (1-to-Many) &rarr; | **booking_id** (PK)<br>*guest_id* (FK)<br>*room_number* (FK)<br>check_in_date | &larr; (Many-to-1) &rarr; | **room_number** (PK)<br>room_type<br>price (CHECK > 0) |

---

## 💻 SQL Implementation Example

Here is how this conceptual hotel schema translates into production-ready SQL code:

```sql
-- 1. Create the Rooms Table
CREATE TABLE rooms (
    room_number INT PRIMARY KEY,
    room_type VARCHAR(50) NOT NULL,
    price_per_night NUMERIC(6, 2) NOT NULL CHECK (price_per_night > 0)
);

-- 2. Create the Guests Table
CREATE TABLE guests (
    guest_id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

-- 3. Create the Bookings Table (The Relational Connector)
CREATE TABLE bookings (
    booking_id SERIAL PRIMARY KEY,
    guest_id INT NOT NULL,
    room_number INT NOT NULL,
    check_in_date DATE NOT NULL,
    
    -- Establishing Foreign Keys (Referential Integrity)
    CONSTRAINT fk_guest 
        FOREIGN KEY(guest_id) REFERENCES guests(guest_id) 
        ON DELETE CASCADE,
        
    CONSTRAINT fk_room 
        FOREIGN KEY(room_number) REFERENCES rooms(room_number)
        ON DELETE RESTRICT
);
