# financial-tracking-program-with-python
import sqlite3

# Create a Transaction class to represent a transaction
class Transaction:
  def __init__(self, date, transaction_type, amount, category=None):
    self.date = date
    self.type = transaction_type
    self.amount = amount
    self.category = category

def create_connection():
  """ Creates a connection to the finance.db database """
  conn = sqlite3.connect('finance.db')
  return conn
def create_table(conn):
  """ Creates a table named 'transactions' in the database """
  cursor = conn.cursor()
  cursor.execute("""CREATE TABLE IF NOT EXISTS transactions (
                      date text,
                      type text,
                      amount real,
                      category text
                  )""")

  cursor.execute("""CREATE TABLE IF NOT EXISTS budgets (
                      id INTEGER PRIMARY KEY AUTOINCREMENT,
                      amount REAL,
                      category TEXT
                  )""")
  conn.commit()

def add_transaction(conn, transaction):
  """ Adds a transaction object to the 'transactions' table """
  cursor = conn.cursor()
  cursor.execute("""INSERT INTO transactions (date, type, amount, category)
                      VALUES (?, ?, ?, ?)""", (transaction.date, transaction.type, transaction.amount, transaction.category))
  conn.commit()

def get_transactions(conn, start_date, end_date):
  """ Retrieves transactions within a specified date range """
  cursor = conn.cursor()
  cursor.execute("""SELECT * FROM transactions WHERE date BETWEEN ? AND ?""", (start_date, end_date))
  return cursor.fetchall()

def set_budget(conn, category, amount):
  """ Stores a budget amount for a specific category """
  # Implement logic to check if budget table exists and create it if necessary
  cursor = conn.cursor()
  cursor.execute("""INSERT INTO budgets (category, amount) VALUES (?, ?)""", (category, amount))
  conn.commit()

def get_budget(conn, category):
  """ Retrieves the budget amount for a specific category """
  cursor = conn.cursor()
  cursor.execute("""SELECT amount FROM budgets WHERE category = ?""", (category,))
  result = cursor.fetchone()
  return result[0] if result else None  # Return budget amount if found, None otherwise

def main():
  # Connect to the database
  conn = create_connection()
  create_table(conn) 

  while True:
    print("1. Add Transaction")
    print("2. View Transactions")
    print("3. Set Budget")
    print("4. Exit")
    choice = input("Enter your choice: ")

    if choice == '1':
      date = input("Enter transaction date (YYYY-MM-DD): ")
      transaction_type = input("Enter transaction type (income/expense): ")
      amount = float(input("Enter transaction amount: "))
      category = input("Enter category (optional): ")

      # Create a transaction object
      transaction = Transaction(date, transaction_type, amount, category)

      # Add transaction to the database
      add_transaction(conn, transaction)
      print("Transaction added successfully!")

    elif choice == '2':
      start_date = input("Enter start date (YYYY-MM-DD): ")
      end_date = input("Enter end date (YYYY-MM-DD): ")

      transactions = get_transactions(conn, start_date, end_date)

      if transactions:
        print("Transactions:")
        for transaction in transactions:
          print(f"Date: {transaction[0]}, Type: {transaction[1]}, Amount: {transaction[2]}, Category: {transaction[3]}")
      else:
        print("No transactions found for the specified date range.")

    elif choice == '3':
      category = input("Enter category for budget: ")
      amount = float(input("Enter budget amount: "))
      set_budget(conn, category, amount)
      print(f"Budget set for {category}: {amount}")

    elif choice == '4':
      print("Exiting...")
      break

    else:
      print("Invalid choice!")

  conn.close()

if __name__== "__main__":
  main()
