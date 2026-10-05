# Expense Tracker (Python + MySQL)

A beginner-friendly command-line application to record, search and analyse personal expenses, built with Python and MySQL.

## Project Overview

This project shows how Python and SQL work together in a real application. Expenses are stored in a MySQL database. The user manages them through a simple terminal menu. The code is split into three layers (user interface, SQL logic, database connection) so it is easy to read, test and extend.

## Key Features

- Add a new expense (date, category, description, amount)
- View all expenses in a formatted table
- Search and filter by date, category or month
- Update and delete expenses (with delete confirmation)
- Monthly summary: total, number of expenses, highest, average, category-wise totals
- Overall summary: total spent, number of expenses, highest, average
- Input validation and error handling
- Parameterized SQL queries to prevent SQL injection
- Database credentials kept out of the code using a `.env` file

## Technologies Used

| Technology | Purpose |
|---|---|
| Python 3 | Application logic and CLI |
| MySQL | Data storage |
| mysql-connector-python | Connects Python to MySQL |
| python-dotenv | Loads database credentials from a `.env` file |

## Python Libraries / Dependencies

```
mysql-connector-python>=8.0.33
python-dotenv>=1.0.0
```

## MySQL Database Information

- Database name: `expense_tracker`
- Table name: `expenses`
- The database and table are created automatically when the app starts. The plain SQL is also available in `schema.sql`.

### Table Structure: `expenses`

| Column | Type | Description |
|---|---|---|
| id | INT, PRIMARY KEY, AUTO_INCREMENT | Unique expense ID |
| expense_date | DATE, NOT NULL | Date of the expense |
| category | VARCHAR(50), NOT NULL | Category such as Food or Rent |
| description | VARCHAR(255), NOT NULL | Short description |
| amount | DECIMAL(10,2), NOT NULL, CHECK (amount > 0) | Amount spent |
| created_at | TIMESTAMP, DEFAULT CURRENT_TIMESTAMP | When the record was created |

An index on `expense_date` speeds up date and month searches.

## How the Application Works

```
User  ->  app.py  ->  expense_manager.py  ->  database.py  ->  MySQL
        (menu and      (SQL queries)         (connection)
         validation)
```

1. `app.py` shows the menu, reads input and validates it.
2. `expense_manager.py` runs the matching SQL query using `%s` placeholders.
3. `database.py` opens the MySQL connection, runs the query and closes the connection.
4. Results come back to `app.py`, which prints them as a table.

## Project Folder Structure

```
expense-tracker-python-mysql/
│
├── app.py                  # CLI menu, input validation, output formatting
├── database.py             # MySQL connection and query helpers
├── expense_manager.py      # All SQL operations (CRUD, filters, summaries)
├── load_sample_data.py     # Inserts 12 sample expenses (run once)
├── schema.sql              # Database and table creation SQL
├── sample_data.sql         # Sample data as plain SQL
├── requirements.txt        # Python dependencies
├── .env.example            # Template for database credentials (no real secrets)
├── .gitignore              # Files excluded from Git
├── README.md               # Project documentation
└── screenshots/
    └── expense-tracker-output.png
```

## Installation Requirements

- Python 3.8 or later
- MySQL Server 8.0 or later (running on your computer)
- pip (comes with Python)
- Git (only needed to clone the repository)

## Step-by-Step Setup

### 1. Clone the repository

```
git clone https://github.com/YOUR_USERNAME/expense-tracker-python-mysql.git
cd expense-tracker-python-mysql
```

### 2. (Recommended) Create a virtual environment

```
python -m venv venv
venv\Scripts\activate          # Windows
source venv/bin/activate       # macOS / Linux
```

### 3. Install dependencies

```
pip install -r requirements.txt
```

### 4. Set up MySQL

Make sure the MySQL service is running. You do not need to create the database manually, because the app creates `expense_tracker` and the `expenses` table on first run. To create them yourself instead:

```
mysql -u root -p < schema.sql
```

### 5. Configure your MySQL username and password securely

Credentials are read from a `.env` file that is **never uploaded to GitHub**.

Copy the template:

```
copy .env.example .env         # Windows
cp .env.example .env           # macOS / Linux
```

Open `.env` and enter your own values:

```
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_mysql_password_here
```

### 6. (Optional) Load sample data, once only

```
python load_sample_data.py
```

### 7. Run the project

```
python app.py
```

## Example CLI Output

```
==============================
       EXPENSE TRACKER
==============================
 1. Add a new expense
 2. View all expenses
 3. Search / filter expenses
 4. Update an expense
 5. Delete an expense
 6. Monthly summary
 7. Overall summary
 8. Exit
------------------------------
Enter your choice (1-8): 6

--- Monthly Summary ---
Enter month (YYYY-MM): 2026-09

===== MONTHLY SUMMARY: 2026-09 =====
Total expenses      : Rs. 13,750.25
Number of expenses  : 7
Highest expense     : Rs. 8,000.00 (Rent - Monthly room rent on 2026-09-01)
Average expense     : Rs. 1,964.32

Category-wise total:
  Rent             Rs.   8,000.00
  Food             Rs.   2,030.50
  Education        Rs.   1,499.00
  Utilities        Rs.   1,120.75
  Entertainment    Rs.     600.00
  Transport        Rs.     500.00
```

```
Enter your choice (1-8): 7

===== OVERALL SUMMARY =====
Total amount spent  : Rs. 24,159.50
Number of expenses  : 12
Highest expense     : Rs. 8,000.00 (Rent - Monthly room rent on 2026-10-01)
Average expense     : Rs. 2,013.29
```

![Expense Tracker output](screenshots/expense-tracker-output.png)

## Sample Expense Data

`load_sample_data.py` (or `sample_data.sql`) inserts these 12 records:

| Date | Category | Description | Amount (Rs.) |
|---|---|---|---|
| 2026-09-01 | Rent | Monthly room rent | 8000.00 |
| 2026-09-03 | Food | Groceries from market | 1250.50 |
| 2026-09-05 | Transport | Metro card recharge | 500.00 |
| 2026-09-09 | Food | Dinner with friends | 780.00 |
| 2026-09-14 | Education | Python course subscription | 1499.00 |
| 2026-09-20 | Entertainment | Movie tickets | 600.00 |
| 2026-09-25 | Utilities | Electricity bill | 1120.75 |
| 2026-10-01 | Rent | Monthly room rent | 8000.00 |
| 2026-10-02 | Food | Weekly groceries | 1340.25 |
| 2026-10-03 | Transport | Bus pass | 450.00 |
| 2026-10-04 | Health | Pharmacy | 320.00 |
| 2026-10-05 | Entertainment | OTT subscription | 299.00 |

## Monthly Summary Explanation

The user enters a month as `YYYY-MM`. The app then:

1. Filters records with `WHERE YEAR(expense_date) = %s AND MONTH(expense_date) = %s`.
2. Calculates the total (`SUM`), count (`COUNT`) and average (`AVG`).
3. Finds the highest expense with `ORDER BY amount DESC LIMIT 1`.
4. Groups spending by category with `GROUP BY category`, highest first.

If the month has no records, the app shows a friendly message instead of zeros.

## CRUD Operations Explanation

| Operation | Menu option | SQL used |
|---|---|---|
| Create | 1. Add a new expense | `INSERT INTO expenses ...` |
| Read | 2. View all / 3. Search | `SELECT ... WHERE ...` |
| Update | 4. Update an expense | `UPDATE expenses SET ... WHERE id = %s` |
| Delete | 5. Delete an expense | `DELETE FROM expenses WHERE id = %s` |

- Update shows the current record. Press Enter to keep any field unchanged.
- Delete asks for confirmation (y/n) before removing the record.
- Update and delete first check that the ID exists.

## Error Handling and Input Validation

- Dates must be in `YYYY-MM-DD` format, months in `YYYY-MM`.
- Amounts must be positive numbers with at most two decimal places.
- Category and description cannot be empty or too long.
- IDs must be positive whole numbers.
- Invalid menu choices are rejected with a clear message.
- Every input re-prompts until the value is valid.
- Database errors are caught (`mysql.connector.Error`) and shown as friendly messages.
- Connections are always closed using `try/finally`.
- All SQL uses `%s` placeholders, so user input can never change the query (SQL injection protection).

## Future Improvements

- User accounts and login (a `users` table with a foreign key)
- A separate `categories` table
- Monthly budget limits with warnings
- Export reports to CSV or Excel
- Charts using matplotlib
- Web interface using Flask or Django
- Unit tests with pytest

## Learning Outcomes

- Connecting Python to MySQL with `mysql-connector-python`
- Writing parameterized SQL queries and understanding SQL injection
- Using SQL aggregate functions (`SUM`, `AVG`, `MAX`, `COUNT`) and `GROUP BY`
- Choosing correct data types (`DECIMAL` for money, `DATE` for dates)
- Structuring code into separate layers
- Validating user input and handling exceptions
- Keeping secrets out of source code with environment variables and `.gitignore`
- Using Git and GitHub

## Author

**Tejashree Hake**
BBA (Computer Applications) student, Savitribai Phule Pune University, Pune

- GitHub: [YOUR_USERNAME](https://github.com/YOUR_USERNAME)
- LinkedIn: [Your Name](https://www.linkedin.com/in/YOUR_LINKEDIN_ID)
- Email: your.email@example.com
